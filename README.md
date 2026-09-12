# ci-workflows

Shared GitHub Actions building blocks for the `lukislp` repositories: the pieces that were
copied into every repo by hand and drifted apart. Callers pin this repo by commit SHA (Dependabot
keeps the pin moving), exactly like any third-party action.

## Reusable workflows (`workflow_call`)

| Workflow | What it does | Caller |
|---|---|---|
| `.github/workflows/dependabot-auto-merge.yml` | Merges Dependabot patch/minor PRs once green; brings every open PR that has auto-merge armed (or was opened by a person) back onto the tip after each push to the default branch, hourly as a safety net. | `templates/callers/dependabot-auto-merge.yml` |
| `.github/workflows/dependabot-lockfiles.yml` | Regenerates NuGet `packages.lock.json` and/or `uv.lock` on Dependabot's own PRs with the CI tool versions and pushes the result back onto the PR branch. | `templates/callers/dependabot-lockfiles.yml` |

Both need `AUTOMERGE_TOKEN` (a fine-grained PAT with contents + pull requests write on the
calling repo): as a repository secret for auto-merge, and additionally as a **Dependabot**
secret for the lock-file refresh (Dependabot-triggered `pull_request` runs only see those).

## Composite actions

| Action | Use |
|---|---|
| `.github/actions/deploy-key-push` | Switch `origin` to SSH with the release deploy key so the following push bypasses the branch ruleset (semantic-release commits, image-tag bumps). |
| `.github/actions/nuget-severity-gate` | `dotnet list package --vulnerable` for a solution, JSON report as artifact, fails on High/Critical (configurable; tolerated packages or advisory ids via `allow` / `allow-advisories`, each justified in the caller). |
| `.github/actions/semantic-release` | Runs semantic-release from a pinned, Dependabot-maintained toolchain (semantic-release + changelog/git/github/exec plugins, lock file in the action directory) with the outputs of the marketplace action (`new_release_published` / `new_release_version` / `new_release_git_tag`); `dry-run: true` for the version-only pass. The `.releaserc.json` stays in the caller. |

## Templates (copied, not called)

`templates/scorecard.yml` is the standard OpenSSF Scorecard workflow. It cannot be a reusable
workflow: `ossf/scorecard-action` publishes results only from a workflow file that lives in the
analysed repository itself.

## Pinning

```yaml
jobs:
  dependabot:
    uses: lukislp/ci-workflows/.github/workflows/dependabot-auto-merge.yml@<sha> # v1.0.0
    secrets:
      AUTOMERGE_TOKEN: ${{ secrets.AUTOMERGE_TOKEN }}
```

```yaml
      - uses: lukislp/ci-workflows/.github/actions/deploy-key-push@<sha> # v1.0.0
        with:
          deploy-key: ${{ secrets.SEMANTIC_RELEASE_DEPLOY_KEY }}
```

```yaml
      - id: semrel
        uses: lukislp/ci-workflows/.github/actions/semantic-release@<sha> # v1.2.0
        with:
          dry-run: true # omit for the real release
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

Releases are tagged `vX.Y.Z`; the `# vX.Y.Z` comment next to the SHA is what Dependabot updates.
Every change here is a pull request, validated by `validate.yml` (YAML parse + job graph check of
every workflow and action) before it can merge.
