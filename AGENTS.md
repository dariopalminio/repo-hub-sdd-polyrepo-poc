# Repository Guidance

This workspace is a Spec Kit hub for a polyrepo proof of concept, not an application source repository. Use [`.meta.yaml`](.meta.yaml) to identify member repositories and their tags. A member directory may be a separate Git repository; inspect its own instructions and Git status before editing, and keep changes within the repository requested by the user.

## Spec-Driven Workflow

- The configured integration is Copilot; see [`.specify/integration.json`](.specify/integration.json).
- The standard workflow is specify, review, plan, review, tasks, implement. Follow [the workflow definition](.specify/workflows/speckit/workflow.yml) and the relevant skill in [`.github/skills/`](.github/skills/).
- The service member's [Spec Kit store config](repo-service-sdd-polyrepo-poc/.specify-store/config.yaml) points to the workspace `specs/` directory. Verify the feature files and target repository exist before assuming they are present.
- The PowerShell helpers are under [`.specify/scripts/powershell/`](.specify/scripts/powershell/). Use `pwsh .specify/scripts/powershell/check-prerequisites.ps1 -Help` and `pwsh .specify/scripts/powershell/create-new-feature.ps1 -Help` for current usage. `check-prerequisites.ps1 -PathsOnly` resolves paths without validating or persisting feature state.
- Script root discovery uses the nearest `.specify/` directory. `SPECIFY_INIT_DIR` must point to an existing project root containing `.specify/`; do not assume the member's `.specify-store/` config makes it a valid target for these scripts.
- No application build or test command is documented in this workspace. Check the target repository for its own project files and test instructions before choosing validation commands.

## Current Documentation Caveats

- The root [README](README.md) contains unresolved merge-conflict markers. Do not treat it as authoritative guidance until it is repaired; do not resolve it as part of unrelated work.
- [The Spec Kit constitution](.specify/memory/constitution.md) is still a template. Do not infer project principles from its example text.