# Grimmory unRAID Runbook

This runbook sets up Grimmory on unRAID against a cloned copy of the rebuilt Calibre library plus a writable BookDrop folder.

## Live Reference Environment

- live Calibre library:
  `/mnt/m2cache/calibre-cleanup-0326/CalibreLibrary-New`
- Calibre-Web:
  `http://100.85.214.86:8083`
- Calibre desktop admin:
  `https://100.85.214.86:8181`
- Calibre metadata DB backups:
  `/mnt/m2cache/calibre-cleanup-0326/Backups/metadata-db`

## One-Writer Rule

Only one system should be treated as the active writer for a given library tree.

During this test:

- Calibre-Web and Calibre desktop remain the writers for the live Calibre library
- Grimmory is the writer for the cloned Grimmory test library

Do not point Grimmory at the live Calibre library for write testing.

## unRAID Folder Layout

Recommended paths for the test:

- Grimmory app data:
  `/mnt/user/appdata/grimmory/data`
- Grimmory MariaDB:
  `/mnt/user/appdata/grimmory-mariadb`
- Grimmory test workspace root:
  `/mnt/user/data/grimmory-test`
- cloned Grimmory test library:
  `/mnt/user/data/grimmory-test/CalibreLibrary-GrimmoryTest`
- BookDrop:
  `/mnt/user/data/grimmory-test/bookdrop`

BookDrop must remain outside the library root to prevent duplicate discovery.

## Container Defaults

Use the same unRAID container user defaults already used by the Calibre stacks:

- `PUID=99`
- `PGID=100`
- `TZ=Europe/London`

Keep Grimmory LAN/Tailscale-only during this phase. Do not expose it publicly without adding a proper reverse proxy and origin controls later.

## Compose File

Use:

- [docker-compose.unraid-calibre-test.yml](/Users/petetreadaway/Projects/grimmory/deploy/compose/docker-compose.unraid-calibre-test.yml)

Before launch:

1. Copy the file to your unRAID stack location if needed.
2. Replace:
   - `CHANGE_ME_GRIMMORY_DB_PASSWORD`
   - `CHANGE_ME_MARIADB_ROOT_PASSWORD`
3. Confirm the library and BookDrop host paths match your unRAID shares.

## Clone Preparation

Prepare the Grimmory test workspace on unRAID:

```bash
mkdir -p /mnt/user/data/grimmory-test
mkdir -p /mnt/user/data/grimmory-test/bookdrop
rsync -aH --delete /mnt/m2cache/calibre-cleanup-0326/CalibreLibrary-New/ /mnt/user/data/grimmory-test/CalibreLibrary-GrimmoryTest/
mkdir -p /mnt/user/appdata/grimmory/data
mkdir -p /mnt/user/appdata/grimmory-mariadb
```

If you want a quicker first pass, create the clone once and then refresh it before each major test cycle.

## Launch

```bash
docker compose -f deploy/compose/docker-compose.unraid-calibre-test.yml up -d
```

## Required Validation

### 1. Container Health

```bash
docker ps
docker logs --tail 100 grimmory
docker logs --tail 100 grimmory-mariadb
curl -fsS http://127.0.0.1:6060/api/v1/healthcheck
```

Expected result:

- MariaDB is healthy
- Grimmory responds on `/api/v1/healthcheck`
- no datasource or path-access errors in logs

### 2. Permission Checks

Confirm the Grimmory container can actually use the mounted paths:

```bash
docker exec grimmory sh -lc 'ls -ld /books /bookdrop && touch /bookdrop/.permcheck && rm /bookdrop/.permcheck'
docker exec grimmory sh -lc 'find /books -maxdepth 1 | head'
```

Before claiming direct-write testing works, also verify create, rename, move, and delete on sacrificial files inside the clone:

```bash
docker exec grimmory sh -lc 'mkdir -p /books/.grimmory-permcheck && touch /books/.grimmory-permcheck/test.tmp && mv -f /books/.grimmory-permcheck/test.tmp /books/.grimmory-permcheck/test-renamed.tmp && rm -f /books/.grimmory-permcheck/test-renamed.tmp && rmdir /books/.grimmory-permcheck'
```

### 3. In-App Setup

1. Open `http://<unraid-host-or-tailscale-ip>:6060`
2. Create the Grimmory admin account
3. Create one library pointing at `/books`
4. Let Grimmory scan the clone

The admin account should be sufficient for BookDrop testing. If you later create a non-admin test user, grant that user BookDrop access explicitly before testing the BookDrop UI.

### 4. Library Validation

Check:

- Grimmory accepts `/books` as a valid library path
- the initial scan completes
- sample EPUB and PDF titles open correctly
- scanned totals are close to the clone's Calibre `metadata.db` counts

Count the clone directly if needed:

```bash
sqlite3 /mnt/user/data/grimmory-test/CalibreLibrary-GrimmoryTest/metadata.db 'select count(*) from books;'
```

### 5. BookDrop Validation

Copy a few small test titles into BookDrop and then validate end to end:

```bash
cp -f /path/to/test1.epub /mnt/user/data/grimmory-test/bookdrop/
cp -f /path/to/test2.pdf /mnt/user/data/grimmory-test/bookdrop/
```

Then in Grimmory:

1. open BookDrop
2. confirm files appear
3. run manual rescan if needed
4. finalize import
5. verify the files now exist in the cloned library, not the live Calibre tree

### 5a. MAM/qBittorrent BookDrop Handoff

The live MAM/qBittorrent handoff has an immediate signal path plus a one-minute catchall:

```bash
docker exec MAM-QBTorrent wget -qO- http://127.0.0.1:18080/api/v2/app/preferences | jq -r '.autorun_enabled, .autorun_program'
cat /boot/config/plugins/dynamix/grimmory-bookdrop-qbit.cron
ps -ef | grep -E 'watch_bookdrop_requests|reconcile_bookdrop_from_qbit' | grep -v grep
```

Expected result:

- qBittorrent autorun is enabled.
- autorun points at `/config/scripts/request_bookdrop_link.sh "%L" "%F" "%I" "%N"`.
- cron contains `@reboot /mnt/user/appdata/MAM-QBTorrent/scripts/watch_bookdrop_requests.sh`, the one-minute reconciler, and the MAM Dynamic Seedbox updater.
- the watcher is running for the current boot.

Validate a linked file without moving or deleting the seeded source:

```bash
stat -c '%d:%i links=%h owner=%u:%g %n' \
  '/mnt/m2cache/MAM-QBTorrent/media/books/<file>' \
  '/mnt/m2cache/grimmory-test/bookdrop/<file>'
docker exec MAM-QBTorrent wget -qO- 'http://127.0.0.1:18080/api/v2/torrents/info?filter=completed' \
  | jq -r '.[] | select(.name=="<torrent name>") | {name,category,state,progress,content_path,hash}'
docker logs --since '10 minutes ago' grimmory 2>&1 | grep -Ei 'bookdrop|<file>|error|warn'
```

Troubleshooting paths:

- Linker log: `/mnt/user/appdata/MAM-QBTorrent/scripts/bookdrop-linker.log`
- MAM Dynamic Seedbox log: `/mnt/user/appdata/MAM-QBTorrent/scripts/mam-dynamic-seedbox.log`
- Request queue: `/mnt/user/appdata/MAM-QBTorrent/scripts/bookdrop-requests.d/`
- Linked markers: `/mnt/user/appdata/MAM-QBTorrent/scripts/bookdrop-linked.d/`
- Pending records: `/mnt/user/appdata/MAM-QBTorrent/scripts/bookdrop-pending.d/`
- Conflict records: `/mnt/user/appdata/MAM-QBTorrent/scripts/bookdrop-conflicts.d/`

Important traps:

- The qBittorrent hook is for speed. The one-minute reconciler is the reliability path and should stay enabled.
- The container hook only writes a request. Hardlinking must happen on the host because Docker cannot hardlink between the separate `/downloads` and `/bookdrop` bind mounts.
- Linked files should land directly in `/mnt/m2cache/grimmory-test/bookdrop`. A persistent `/bookdrop/books` directory can hide new files from Grimmory's immediate watcher.
- Keep `/mnt/m2cache/MAM-QBTorrent` and `/mnt/m2cache/grimmory-test/bookdrop` on the same filesystem/cache device. unRAID mover or share/cache changes can break hardlinking.
- Marker files prove only that the handoff linked into BookDrop. They do not prove Grimmory finalized the import.
- MouseSearch runs on OCI while `MAM-QBTorrent` announces from unRAID. If MAM reports `Unrecognized host/PassKey`, run `/mnt/user/appdata/MAM-QBTorrent/scripts/update_mam_dynamic_seedbox.sh` on unRAID and reannounce the torrent. Do not fix this by enabling MouseSearch's OCI-side Dynamic IP Updater unless MouseSearch and qBittorrent share the same public egress IP.

### 6. Restart Validation

```bash
docker compose -f deploy/compose/docker-compose.unraid-calibre-test.yml restart
```

Recheck:

- health endpoint
- library presence
- BookDrop status
- qBittorrent autorun preference
- request watcher process
- catchall cron entry
- recent logs

## Recommended Smoke Tests

- open one EPUB
- open one PDF
- import at least two files via BookDrop
- perform one rename/move/delete-style operation on sacrificial files in the clone
- rescan and confirm Grimmory remains consistent afterward

## Rollback

If the test misbehaves:

1. stop Grimmory and MariaDB
2. discard the cloned test library
3. recreate the clone from the live Calibre source
4. clear test BookDrop files if needed
5. restart the stack

The live Calibre library should not need restoration if Grimmory was kept on the clone.
