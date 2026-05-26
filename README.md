# pr-author-limit

GitHub Action that auto-closes new pull requests when the author already has more than N open PRs in the same repository.

Particularly useful for tackling **AI-generated PRs** that can flood a repository with concurrent submissions faster than a human can review them.

## What it does

When a PR is opened or reopened:

1. Counts how many PRs the same author currently has open.
2. If the count exceeds the configured limit, the action:
   - Posts a comment on the new PR explaining why it's being closed.
   - Closes the PR.

Existing open PRs from that author are left alone: only the new one is closed.

## Usage

Create `.github/workflows/pr-author-limit.yml` in your repository:

```yaml
name: PR author limit

on:
  pull_request_target:
    types: [opened, reopened]

permissions:
  pull-requests: write

jobs:
  enforce:
    runs-on: ubuntu-latest
    steps:
      - uses: mirkoalicastro/pr-author-limit@v1
        with:
          max-open-prs: 2
```

The workflow uses `pull_request_target`, so it must exist on the default branch to take effect. Changes inside a PR do not run until merged.

## Inputs

| Name           | Required | Default        | Description                                                                 |
|----------------|----------|----------------|-----------------------------------------------------------------------------|
| `max-open-prs` | no       | `2`            | Maximum number of open PRs allowed per author. New PRs that exceed are closed. |
| `github-token` | no       | `${{ github.token }}` | Token used to list and close PRs. Needs `pull-requests: write`.       |

## Configurable limit via repository variable

To change the limit without editing the workflow, wire it to a repo variable:

```yaml
- uses: mirkoalicastro/pr-author-limit@v1
  with:
    max-open-prs: ${{ vars.MAX_OPEN_PRS_PER_AUTHOR || '2' }}
```

Then create the variable under **Settings → Secrets and variables → Actions → Variables → New repository variable**:

- Name: `MAX_OPEN_PRS_PER_AUTHOR`
- Value: any non-negative integer

No workflow restart is needed. The next PR opened will pick up the new value.

## Permissions

The workflow needs `pull-requests: write` on the `GITHUB_TOKEN`. No PAT or secret is required.

`pull_request_target` runs in the context of the base branch, so the token has write access even for PRs from forks. The action does not check out PR code, so there is no risk of executing untrusted code from a fork.

## Behavior notes

- Triggers: `opened` and `reopened` PRs only. Existing PRs are not affected retroactively.
- The author count includes the newly opened PR itself.
- All open PRs are fetched via `github.paginate` (100 per page, no hard cap). Note GitHub's REST API limits `per_page` to 100, so pagination is mandatory above that.
- Bot accounts (e.g. `dependabot[bot]`, `github-actions[bot]`) are subject to the same limit. Add an early-return filter in a wrapper step if you want to exempt specific accounts.

## License

See [LICENSE](LICENSE).
