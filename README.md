# ci-workflows

Shared GitHub Actions building blocks for the `lukislp` repositories: the pieces that were
copied into every repo by hand and drifted apart. Callers pin this repo by commit SHA (Dependabot
keeps the pin moving), exactly like any third-party action.

## Reusable workflows (`workflow_call`)

| Workflow | What it does | Caller |
|---|---|---|
| `.github/workflows/dependency-review.yml` | Fails a pull request that introduces a dependency with a known advisory, by comparing the base and head dependency graphs - the gap Dependabot alerts only close after the merge. Public repositories only (the graph comparison needs GHAS on private repos). Inputs `fail-on-severity`, `comment-summary-in-pr`, `allow-ghsas`, `deny-licenses`. | `templates/callers/dependency-review.yml` |
| `.github/workflows/hacs-ci.yml` | The whole pipeline of a Home Assistant custom integration: HACS + hassfest validation, ruff (check + format), pytest with coverage badge, Atheris fuzz pass. Releases are not part of it: they go through the lktec-bot release module. Inputs `component` and `fuzz-script`; `release`, `release-group` and `default-branch` are deprecated and ignored, accepted for one more minor version so that existing callers keep working. | `templates/callers/hacs-ci.yml` |

The Dependabot auto-merge and lock-file reusables were retired; both are now the `automerge` and `lockfiles` modules of the lktec-bot GitHub App (see [lukislp/lktec-bot](https://github.com/lukislp/lktec-bot), `docs/modules/automerge.md` and `docs/modules/lockfiles.md`).


## Composite actions

| Action | Use |
|---|---|
| `.github/actions/nuget-severity-gate` | `dotnet list package --vulnerable` for a solution, JSON report as artifact, fails on High/Critical (configurable; tolerated packages or advisory ids via `allow` / `allow-advisories`, each justified in the caller). |

## Templates (copied, not called)

`templates/scorecard.yml` is the standard OpenSSF Scorecard workflow. It cannot be a reusable
workflow: `ossf/scorecard-action` publishes results only from a workflow file that lives in the
analysed repository itself.

## Pinning

Releases are tagged `vX.Y.Z`; the `# vX.Y.Z` comment next to the SHA is what Dependabot updates.
Every change here is a pull request, validated by `validate.yml` (YAML parse + job graph check of
every workflow and action) before it can merge.
