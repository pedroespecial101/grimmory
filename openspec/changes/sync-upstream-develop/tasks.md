## 1. Prepare Sync Context

- [x] 1.1 Confirm the live upstream `develop` tip and compare it with the fork `develop` and docs branch tips.
- [x] 1.2 Verify that the local-only branch delta is limited to Beads metadata, deployment files, and operational docs rather than Grimmory application source files.
- [x] 1.3 Create explicit safety branches for the current local `develop` tip and the current docs/deploy branch tip.

## 2. Rewrite Develop

- [x] 2.1 Add or refresh the `upstream` remote and fetch the source Grimmory branches.
- [x] 2.2 Move local `develop` to `upstream/develop`.
- [x] 2.3 Force-update the fork `develop` branch to the same upstream commit.

## 3. Verify And Record

- [x] 3.1 Verify the final branch tips, tracking state, and working tree status after the rewrite.
- [x] 3.2 Update the OpenSpec tasks to reflect the completed sync work.
