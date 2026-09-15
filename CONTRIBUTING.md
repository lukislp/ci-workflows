# Contributing to ci-workflows

Thanks for taking the time. This repository holds the reusable GitHub Actions workflows and
composite actions shared by the `lukislp` repositories. It contains no application code, so a
change here lands in every pipeline that pins it.

## How changes get in

1. Open an issue first for anything bigger than a typo, so the direction can be agreed before you
   spend time on it.
2. Fork the repository (or branch, if you have write access) and make your change on a branch.
3. Open a pull request against `main`.
4. `main` is protected: a PR merges only after `validate` and `review / dependency-review` are
   green and the branch is up to date with `main` (enable auto-merge and it lands on its own once
   that is the case). Nobody pushes to `main` directly, not even the maintainer.

## What a pull request needs

- **Conventional Commits.** Commit and pull-request titles follow Conventional Commits
  (`feat:`, `fix:`, `docs:`, `ci:`, and so on) - the same convention the consuming repositories
  use, where it drives semantic-release. Squash-merge keeps the PR title as the commit message.
- **Green required checks.** `validate` and `review / dependency-review` are required; a red one
  blocks the merge. [`.github/workflows/validate.yml`](.github/workflows/validate.yml) parses every
  `*.yml` under `.github/` and `templates/` and fails if a file is not valid YAML, if an external
  `uses:` is not pinned to a full 40-character commit SHA, or if a `needs:` entry names a job that
  does not exist in the same file.
- **Pin every action by SHA.** New or bumped `uses:` references carry the full commit SHA with the
  human-readable version as a trailing comment, for example
  `uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1`. The comment is what
  Dependabot updates; the SHA is what `validate` enforces. Local `./`-prefixed references are
  exempt, and the literal `@SHA` placeholder is allowed only inside `templates/`.
- **Keep the caller templates in sync.** `templates/callers/` holds copy-paste stubs for the
  reusable workflows. If you change a workflow's inputs, update the matching template in the same
  pull request. `templates/scorecard.yml` is copied rather than called, because
  `ossf/scorecard-action` only publishes results from a workflow file that lives in the analysed
  repository.
- **Say what breaks.** A changed or removed input is a breaking change for every caller. List the
  affected repositories in the PR body and how they need to be updated.

## Repository layout

- `.github/workflows/` - the reusable workflows (`workflow_call`): `dependabot-auto-merge.yml`,
  `dependabot-lockfiles.yml`, `dependency-review.yml`, `hacs-ci.yml`, plus this repository's own
  `scorecard.yml` and `validate.yml`.
- `.github/actions/` - the composite actions: `deploy-key-push`, `nuget-severity-gate`,
  `semantic-release`.
- `templates/` - caller stubs and the copied Scorecard workflow.

## Trying a change before it is tagged

There is no test suite here; `validate` is the gate. Beyond it, point one consuming repository's
workflow at your branch's commit SHA and let that repository's pipeline run before you tag.

## Releasing

Releases here are tagged by hand - there is no `.releaserc.json` and no release workflow. After a
change merges, push a new `vX.Y.Z` tag following semantic versioning: a new or changed input is a
minor, a removed or renamed one a major. Consumers resolve the commit SHA and keep the
`# vX.Y.Z` comment next to it, so the tag is documentation for humans and Dependabot rather than
the thing resolved at run time.

## Security issues

Please do not open a public issue for a vulnerability - use the private reporting path described
in [SECURITY.md](SECURITY.md). The [Code of Conduct](CODE_OF_CONDUCT.md) applies to every
interaction in this repository.
