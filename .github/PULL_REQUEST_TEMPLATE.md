## What and why

<!-- One or two sentences: what changes, and why. Link the issue if there is one. -->

## Blast radius

<!-- Which reusable workflow or composite action does this touch, and which repositories call it?
     Everything here runs inside other repositories' pipelines, so a change that looks local is not. -->

## Checklist

- [ ] Conventional Commit title (`fix:`, `feat:`, `ci:`, `docs:`, ...)
- [ ] Every `uses:` pinned to a full 40-character commit SHA, or a `./` path within this repository
- [ ] A file other repositories call by SHA still declares `workflow_call` - adding triggers to it turns it into a caller and breaks every consumer
- [ ] `templates/callers/` updated if the inputs or secrets of a reusable workflow changed
- [ ] No secrets, tokens or personal data in code, comments or logs

## After merging

<!-- Versions here are tagged by hand, they are not cut by semantic-release. If consumers need this
     change, say which tag it should go into and which repositories have to move their pin. -->
