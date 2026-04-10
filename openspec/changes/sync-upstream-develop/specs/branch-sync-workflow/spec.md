## ADDED Requirements

### Requirement: Develop branch SHALL mirror upstream Grimmory source
The repository SHALL support resetting the local and fork `develop` branches to the current `grimmory-tools/grimmory:develop` commit when local-only operational work has been kept on `develop`.

#### Scenario: Sync local develop to upstream
- **WHEN** the upstream Grimmory `develop` tip is fetched
- **THEN** local `develop` SHALL be moved to that exact upstream commit

#### Scenario: Sync fork develop to upstream
- **WHEN** local `develop` has been aligned to upstream
- **THEN** the fork's `develop` branch SHALL be force-updated to the same commit

### Requirement: Pre-sync state SHALL be preserved before branch rewrite
Before rewriting `develop`, the repository SHALL preserve the existing local `develop` state and any branch containing local operational work on explicit safety branches.

#### Scenario: Preserve local develop history
- **WHEN** `develop` is about to be rewritten
- **THEN** the current local `develop` tip SHALL be saved on a named safety branch

#### Scenario: Preserve docs branch history
- **WHEN** a docs or deploy branch contains local-only operational commits
- **THEN** that branch tip SHALL remain available after the `develop` rewrite

### Requirement: Branch-local delta SHALL be validated as non-application work
The sync procedure SHALL confirm that the preserved local-only commits do not modify Grimmory application source files before `develop` is rewritten.

#### Scenario: Validate preserved changes
- **WHEN** the local-only branch delta is inspected from the previous shared base
- **THEN** the changed paths SHALL be limited to operational documentation, deploy files, agent guidance, or Beads metadata rather than backend or frontend application code
