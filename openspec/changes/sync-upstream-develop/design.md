## Context

This repo's `develop` branch on the fork was allowed to accumulate local Beads setup and unRAID deployment documentation while the source Grimmory repository continued advancing upstream. The result is that the fork's `develop` no longer acts as a clean mirror of the application source branch, which makes future upstream syncs noisier and obscures whether local changes affect Grimmory code or only local operations.

The current branch-local work needs to be preserved, but it does not belong on the shared integration branch. The sync procedure therefore needs to separate "rewrite `develop` to upstream" from "keep local deployment and Beads notes accessible".

## Goals / Non-Goals

**Goals:**
- Preserve the current local `develop` and docs/deploy branch state before rewriting anything.
- Realign local `develop` with `grimmory-tools/grimmory:develop`.
- Realign the fork's `develop` branch with the same upstream tip.
- Keep the unRAID/docs work on a separate branch so it can be rebased or revisited later.
- Verify that the branch-local delta does not include Grimmory application source files.

**Non-Goals:**
- Editing Grimmory backend or frontend source code.
- Rebasing or cleaning up the docs/deploy branch beyond making sure it remains available.
- Changing production infrastructure or deployment state outside the git repository.

## Decisions

### Preserve the pre-sync state on explicit safety branches
The sync will create safety refs before rewriting `develop` so the current state is recoverable without reflog archaeology. This is preferable to relying on unstated local history because the worktree already contains a local-only `develop` commit and a docs branch that are meaningful to keep.

Alternative considered: rely on reflog only. Rejected because the user explicitly wants to rewrite `develop`, and explicit safety branches make rollback and handoff clearer.

### Treat upstream `develop` as the source of truth
The target branch state will be the live `grimmory-tools/grimmory:develop` tip, not the fork's current `origin/develop`. This ensures the local checkout and the fork both return to tracking Grimmory source history directly.

Alternative considered: merge upstream into the fork's current `develop`. Rejected because the local-only Beads and docs commits do not belong on the application integration branch.

### Keep deployment/docs work off `develop`
The unRAID and Beads changes will remain accessible on side branches rather than being replayed automatically onto `develop`. This preserves the work without forcing repo-local operational content onto the source-tracking branch.

Alternative considered: rebase the docs branch immediately onto the refreshed `develop`. Deferred because the primary user goal is to update `develop`; replaying the docs work is a separate choice.

## Risks / Trade-offs

- [Force-updating fork `develop` can surprise other clones] → Create safety branches first and report the exact new refs at the end.
- [The docs branch may later conflict with upstream changes in `.gitignore` or `AGENTS.md`] → Keep the docs branch separate and rebase it explicitly only when needed.
- [Untracked local directories may distract from the resulting git status] → Do not delete `.codex/` or `openspec/`; report them clearly as remaining untracked local state.

## Migration Plan

1. Fetch the live upstream Grimmory refs.
2. Create safety branches for the current local `develop` and current docs branch tip.
3. Reset local `develop` to `upstream/develop`.
4. Force-push the fork's `develop` branch to the same commit with `--force-with-lease`.
5. Verify branch tips and working tree state after the rewrite.
6. Leave the docs/deploy branch intact for later rebase or archival decisions.

## Open Questions

- Whether the docs/deploy branch should eventually be rebased onto the new `develop` or archived as a local-ops branch only.
