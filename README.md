# pr-author-limit

GitHub Action that auto-closes new pull requests when the author already has more than N open PRs in the same repository.

Particularly useful for tackling **AI-generated PRs** that can flood a repository with concurrent submissions faster than a human can review them.

## What it does

- Closes the new PR (not the existing ones) when an author exceeds the configured limit.
- Posts a single explanatory comment, deduplicated across reopen events via an HTML marker.
- Skips trusted authors (`OWNER` / `MEMBER` / `COLLABORATOR`) by default; opt out with `skip-trusted-authors: 'false'`.
- Optional comma-separated bot allowlist for accounts like `renovate[bot]` and `dependabot[bot]`.
- Counts open PRs via the GitHub search API (one request) instead of paginating the full PR list.
- Runs on Node.js 24 via `actions/github-script@v9` (SHA-pinned).

## Inputs

| Name                  | Default               | Description                                                                     |
|-----------------------|-----------------------|---------------------------------------------------------------------------------|
| max-open-prs          | 5                     | Maximum open PRs allowed per author.                                            |
| bot-allowlist         | <empty>               | Comma-separated bot logins to skip (example: `renovate[bot],dependabot[bot]`).  |
| skip-trusted-authors  | true                  | When true, skip PRs from owners, members, collaborators.                        |
| github-token          | `${{ github.token }}` | Token used to list and close PRs. Needs `pull-requests: write`.                 |

## Usage

Create `.github/workflows/pr-author-limit.yml` in your repository:

```yaml
name: PR author limit

on:
  pull_request_target:
    types: [opened, reopened]

permissions:
  contents: read

concurrency:
  group: pr-author-limit-${{ github.event.pull_request.user.login }}
  cancel-in-progress: false

jobs:
  enforce:
    runs-on: ubuntu-latest
    timeout-minutes: 5
    permissions:
      pull-requests: write
    steps:
      - uses: mirkoalicastro/pr-author-limit-action@v2
        with:
          max-open-prs: 5
          bot-allowlist: 'renovate[bot],dependabot[bot]'
```

Trusted authors are skipped by default. Set skip-trusted-authors: 'false' to enable the check for trusted authors.

## License

See [LICENSE](LICENSE).
