## Why

The local Grimmory checkout has drifted from the source repository's `develop` branch because the fork's `develop` branch was used to hold repo-local Beads setup and deployment documentation. This makes routine source updates harder than they need to be and mixes upstream application history with local operational notes.

## What Changes

- Reset local and fork `develop` to match `grimmory-tools/grimmory:develop`.
- Preserve the current local-only Beads setup and deployment documentation on non-`develop` safety branches before rewriting `develop`.
- Verify that the preserved branch-local work is limited to Beads metadata, `AGENTS.md`, deploy compose files, and operational docs rather than Grimmory application code.
- Document the exact branch-sync procedure so future upstream syncs can be repeated safely.

## Capabilities

### New Capabilities
- `branch-sync-workflow`: Defines how this repo preserves local operational work while keeping `develop` aligned with the upstream Grimmory source branch.

### Modified Capabilities

## Impact

- Git remotes and branch refs for the local checkout and fork.
- OpenSpec artifacts under `openspec/changes/sync-upstream-develop/`.
- No Grimmory application source behavior is expected to change in `booklore-api/` or `frontend/`.
