# Repository Guidelines

## Agent Workflow

- Start from the root `Justfile` unless you have a clear reason to drop into a subproject.
- Prefer `just api ...` and `just ui ...` over ad hoc commands.
- Keep changes scoped to the relevant project instead of mixing backend, frontend, deployment, and release edits in one pass.
- Target `develop`, not `main`.
- First prove the change with targeted tests around the edited surface, then run the wider suite for that surface before handing off.
- If you cannot run the wider suite, say exactly what you ran, what you skipped, and why.

## Project Structure

- **Backend (`booklore-api/`)**
  - Application code: `src/main/java`
  - Resources and Flyway migrations: `src/main/resources`
  - Tests: `src/test/java`
- **Frontend (`frontend/`)**
  - Application code: `src/app/{core,features,shared}`
  - Translations: `src/i18n/`
  - Assets: `src/assets/`

## Ownership Boundaries

- `deploy/`, `packaging/`, `tools/`, `docs/`, and `assets/` are support surfaces. Do not change them unless the task actually touches deployment, packaging, release automation, or shared docs/assets.
- Keep backend, frontend, deployment, and release work separated unless the task genuinely crosses those boundaries.
- When a change spans multiple surfaces, validate each one explicitly.

## Command Surface

Use these first:

```bash
just check          # run backend + frontend verification
just test           # run backend + frontend tests
just api run        # start Spring Boot with the dev profile
just api test       # run backend tests
just ui dev         # start the Angular dev server
just ui check       # run frontend verification
```

## Backend Rules

- Use 4-space indentation and match surrounding Java style.
- Prefer constructor injection via Lombok patterns already used in the codebase. Do not introduce `@Autowired` field injection.
- Use MapStruct for entity/DTO mapping.
- Keep JPA entities on the `*Entity` suffix.
- Add Flyway migrations as new files named `V<number>__<Description>.sql`.
- Do not edit released migrations in place.
- Prefer focused unit tests; use `@SpringBootTest` only when the Spring context is required.

## Frontend Rules

- Use 2-space indentation in TypeScript, HTML, and SCSS.
- Keep Angular code on standalone components. Do not add NgModules.
- Prefer `inject()` over constructor injection.
- Follow `frontend/eslint.config.js`: component selectors use `app-*`, directive selectors use `app*`, and `any` is disallowed.
- Put user-facing strings in Transloco files under `frontend/src/i18n/`.
- Keep responsive behavior intact.
- Use Vitest for tests.

## Validation

- Use staged verification: prove the behavior locally with targeted tests first, then run the wider suite for that surface.
- Typical backend path: `just api test` and then `just api check`.
- Typical frontend path: targeted Vitest coverage for the changed area and then `just ui check`.
- If the change crosses frontend and backend boundaries, finish with a repo-level pass such as `just test` or `just check`.
- Do not claim completion from a narrow test when a broader runnable suite exists.

## PR Expectations

- If UI behavior changes, capture screenshots or a short recording for the PR.
- PRs in this repo are expected to link an approved issue and include local test output.
- Follow Conventional Commits with scopes, for example `feat(devex): ...` or `fix(entrypoint): ...`.

## unRAID Calibre Deployment Notes

- Current Grimmory primary library: `/mnt/m2cache/grimmory-test/CalibreLibrary-GrimmoryTest`
- Retained legacy Calibre library for rollback only: `/mnt/m2cache/calibre-cleanup-0326/CalibreLibrary-New`
- Direct LAN SSH fallback for the unRAID host: `ssh root@192.168.1.101`
- Retained Calibre access during rollback window:
  - Calibre-Web: `http://100.85.214.86:8083`
  - Calibre desktop admin: `https://100.85.214.86:8181`
- Existing unRAID container defaults from the live Calibre stacks:
  - `PUID=99`
  - `PGID=100`
  - `TZ=Europe/London`
- Grimmory is now the source of truth for the primary library on unRAID.
- Do not write to the retained legacy Calibre tree during the rollback window.
- Grimmory BookDrop remains a sibling of the primary library, not inside the library root:
  - `/mnt/m2cache/grimmory-test/bookdrop`
- Do not design or deploy a steady-state shared-write setup between Calibre and Grimmory.
- MAM/qBittorrent ingestion remains part of the primary Grimmory flow:
  - seeding root: `/mnt/m2cache/MAM-QBTorrent`
  - new-completion hardlinks into `/mnt/m2cache/grimmory-test/bookdrop`
- If rollback is required, re-enable exactly one Calibre-side writer against the retained legacy tree:
  - Calibre-Web
  - full Calibre desktop
  - `calibredb` or tag refresh scripts
  - enrichment or recovery scripts
- The rebuilt-library handover reported:
  - total books: `3576`
  - books with ISBN: `2287`
  - books still without ISBN: `1289`
- Treat those counts as reference points for validation, not exact Grimmory acceptance criteria.
- `AGENTS.md` already has local user edits in this worktree. Merge carefully and do not overwrite unrelated changes.

<!-- BEGIN BEADS INTEGRATION v:1 profile:full hash:f65d5d33 -->
## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Dolt-powered version control with native sync
- Agent-optimized: JSON output, ready work detection, discovered-from links
- Prevents duplicate tracking systems and confusion

### Quick Start

**Check for ready work:**

```bash
bd ready --json
```

**Create new issues:**

```bash
bd create "Issue title" --description="Detailed context" -t bug|feature|task -p 0-4 --json
bd create "Issue title" --description="What this issue is about" -p 1 --deps discovered-from:bd-123 --json
```

**Claim and update:**

```bash
bd update <id> --claim --json
bd update bd-42 --priority 1 --json
```

**Complete work:**

```bash
bd close bd-42 --reason "Completed" --json
```

### Issue Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default, nice-to-have)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

### Workflow for AI Agents

1. **Check ready work**: `bd ready` shows unblocked issues
2. **Claim your task atomically**: `bd update <id> --claim`
3. **Work on it**: Implement, test, document
4. **Discover new work?** Create linked issue:
   - `bd create "Found bug" --description="Details about what was found" -p 1 --deps discovered-from:<parent-id>`
5. **Complete**: `bd close <id> --reason "Done"`

### Quality
- Use `--acceptance` and `--design` fields when creating issues
- Use `--validate` to check description completeness

### Lifecycle
- `bd defer <id>` / `bd supersede <id>` for issue management
- `bd stale` / `bd orphans` / `bd lint` for hygiene
- `bd human <id>` to flag for human decisions
- `bd formula list` / `bd mol pour <name>` for structured workflows

### Auto-Sync

bd automatically syncs via Dolt:

- Each write auto-commits to Dolt history
- Use `bd dolt push`/`bd dolt pull` for remote sync
- No manual export/import needed!

### Important Rules

- ✅ Use bd for ALL task tracking
- ✅ Always use `--json` flag for programmatic use
- ✅ Link discovered work with `discovered-from` dependencies
- ✅ Check `bd ready` before asking "what should I work on?"
- ❌ Do NOT create markdown TODO lists
- ❌ Do NOT use external issue trackers
- ❌ Do NOT duplicate tracking systems

For more details, see README.md and docs/QUICKSTART.md.

## Session Completion

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd dolt push
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds

<!-- END BEADS INTEGRATION -->
