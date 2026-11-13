# escalay/.github

Shared GitHub Actions workflows and defaults for escalay repositories: the checks every repository repeats, written
once. Each is a reusable workflow (`on: workflow_call`) with one job.

| Workflow | Job | What it does |
|---|---|---|
| [`workflows.yml`](.github/workflows/workflows.yml) | `lint` | actionlint and zizmor over the caller's `.github`; input `also`: more folders laid out like `.github` (a `workflows/` folder, perhaps a `dependabot.yml`), separated by spaces |
| [`secrets.yml`](.github/workflows/secrets.yml) | `gitleaks` | gitleaks over the caller's whole history, with the caller's `.gitleaks.toml` when it has one |
| [`title.yml`](.github/workflows/title.yml) | `conventional` | the pull request's title is a Conventional Commit; input `types`: the allowed types, separated by `\|` (default `feat\|fix\|perf\|docs\|refactor\|chore\|test\|build\|ci\|revert`) |
| [`required-statuses.yml`](.github/workflows/required-statuses.yml) | `statuses` | after a dispatched run passed, sets every check the rules for `main` require as a commit status, so a release pull request opened by a workflow can auto-merge; input `branch` (default `main`) |

## Calling them

Pin a release by its commit, with the version in a comment; Dependabot's `github-actions` ecosystem moves the pin when
a new version is tagged.

```yaml
permissions:
  contents: read

jobs:
  workflows:
    uses: escalay/.github/.github/workflows/workflows.yml@<sha> # vX.Y.Z
  secrets:
    uses: escalay/.github/.github/workflows/secrets.yml@<sha> # vX.Y.Z
  title:
    if: github.event_name == 'pull_request'
    uses: escalay/.github/.github/workflows/title.yml@<sha> # vX.Y.Z
  # Only with a release workflow that dispatches CI on its release pull request
  dispatched:
    if: github.event_name == 'workflow_dispatch'
    needs: [workflows, secrets] # and every other job that must pass
    permissions:
      statuses: write
    uses: escalay/.github/.github/workflows/required-statuses.yml@<sha> # vX.Y.Z
```

Pin the latest release by its commit, its version in the comment (Dependabot's `github-actions` updates move both):
`v=$(gh release view -R escalay/.github --json tagName -q .tagName); gh api repos/escalay/.github/commits/$v --jq .sha`
gives the commit of `$v`.

## Check names

A job that calls a reusable workflow reports its check as `<caller job> / <called job>`: the calls above report
`workflows / lint`, `secrets / gitleaks`, `title / conventional` and `dispatched / statuses`. Branch rules require
those names. `npx @escalay/kit repo apply` reads the called workflows and requires the right ones for the jobs that
always run; `required-statuses.yml` reads them back from the rules, so neither needs editing when a name changes.

## Changing this repository

Pull requests titled as Conventional Commits, squash-merged once CI is green. release-please keeps a release pull
request open; merging it tags `vX.Y.Z` and publishes the release. A change to a workflow's inputs, job ids or
behaviour that would break a caller is a `feat!:` or `fix!:` (a new major version).

MIT licensed.
