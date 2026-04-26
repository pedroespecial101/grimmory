# Grimmory on unRAID: Overview

This setup has now been promoted from a controlled adoption test to the primary Grimmory library on unRAID.

The goal is still not to make Grimmory and Calibre share one live library forever. Grimmory is now the active system of record for the primary library, and the old Calibre surfaces are retained only as a short rollback reference.

## Current Primary State

- Grimmory runs against the primary library at `/mnt/m2cache/grimmory-test/CalibreLibrary-GrimmoryTest`
- Grimmory imports new content through BookDrop at `/mnt/m2cache/grimmory-test/bookdrop`
- MAM/qBittorrent seeds from `/mnt/m2cache/MAM-QBTorrent` and hardlinks newly completed torrents into BookDrop
- Calibre-Web and Calibre desktop are no longer active writers for the primary library

## What We Are Not Doing

- We are not designing a permanent shared-write setup between Calibre and Grimmory
- We are not writing to the retained legacy Calibre tree during the rollback window

## Source Of Truth

The current primary Grimmory library lives at:

- `/mnt/m2cache/grimmory-test/CalibreLibrary-GrimmoryTest`

The retained legacy Calibre library for rollback only lives at:

- `/mnt/m2cache/calibre-cleanup-0326/CalibreLibrary-New`

Relevant operating notes came from:

- `/Users/petetreadaway/Projects/new-calibre-library-import/docs/calibre-rebuild.md`
- `/Users/petetreadaway/Projects/new-calibre-library-import/docs/calibre-web.md`
- `/Users/petetreadaway/Projects/new-calibre-library-import/docs/calibre-remaining-metadata-handover.md`

## Active Layout

Use the existing Grimmory workspace on unRAID as the primary layout:

- primary Grimmory library:
  `/mnt/m2cache/grimmory-test/CalibreLibrary-GrimmoryTest`
- BookDrop folder:
  `/mnt/m2cache/grimmory-test/bookdrop`

Keep BookDrop as a sibling of the cloned library, not inside the library root. That avoids double-discovery where Grimmory could see the same files both as library content and as pending BookDrop imports.

## MAM/qBittorrent BookDrop Handoff

The live MAM flow uses two paths so imports are fast but still recoverable:

- qBittorrent runs `/config/scripts/request_bookdrop_link.sh "%L" "%F" "%I" "%N"` when a torrent finishes.
- That container hook only writes a request file into `/config/scripts/bookdrop-requests.d/`.
- The host watcher `/mnt/user/appdata/MAM-QBTorrent/scripts/watch_bookdrop_requests.sh` polls those requests every two seconds and runs the host hardlinker.
- The catchall reconciler `/mnt/user/appdata/MAM-QBTorrent/scripts/reconcile_bookdrop_from_qbit.sh` runs every minute from `/boot/config/plugins/dynamix/grimmory-bookdrop-qbit.cron`.
- The host hardlinker is `/mnt/user/appdata/MAM-QBTorrent/scripts/link_completed_to_bookdrop_host.sh`.
- The MAM Dynamic Seedbox updater `/mnt/user/appdata/MAM-QBTorrent/scripts/update_mam_dynamic_seedbox.sh` also runs from unRAID cron so MAM sees the same egress path as qBittorrent.

The actual hardlink must happen on the host paths. Inside the qBittorrent container, `/downloads` and `/bookdrop` are separate bind mounts, and hardlinks across those mounts fail with `Cross-device link` even though both host paths live under `/mnt/m2cache`.

Linked files are placed directly under `/mnt/m2cache/grimmory-test/bookdrop`, not under a persistent `books/` subfolder. Grimmory watches the BookDrop root and only scans nested content immediately when a new top-level directory appears; putting new files under an already-existing category folder can delay detection until a periodic rescan.

MouseSearch itself runs on OCI. Its Dynamic IP Updater must not be used for the live unRAID `MAM-QBTorrent` client unless both services share the same public egress IP. If MAM reports `Unrecognized host/PassKey`, check `/mnt/user/appdata/MAM-QBTorrent/scripts/mam-dynamic-seedbox.log` and update from unRAID.

## Retained Legacy Calibre Services

The old Calibre interfaces are retained only for rollback:

- Calibre-Web: `http://100.85.214.86:8083`
- Calibre desktop admin: `https://100.85.214.86:8181`

Those services point at the retained legacy Calibre library and should remain stopped unless rollback work is required.

## Operational Expectations

- Grimmory remains the only active writer for the primary library
- BookDrop ingest stays one-way from qBittorrent into Grimmory
- qBittorrent seed files are never moved or deleted by the handoff scripts
- BookDrop marker files mean linked into BookDrop, not successfully imported into Grimmory
- The retained Calibre tree is used only if rollback becomes necessary
- The rollback window should expire before any permanent deletion of the retained Calibre surfaces

## Next Docs

- Technical runbook: [grimmory-unraid-runbook.md](/Users/petetreadaway/Projects/grimmory/docs/grimmory-unraid-runbook.md)
- Cutover path: [grimmory-cutover.md](/Users/petetreadaway/Projects/grimmory/docs/grimmory-cutover.md)
- Private environment details: [README.md](/Users/petetreadaway/Projects/grimmory/docs/private/README.md)
