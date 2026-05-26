# pr-author-limit

GitHub Action that auto-closes new pull requests when the author already has more than N open PRs in the same repository.

Particularly useful for tackling **AI-generated PRs** that can flood a repository with concurrent submissions faster than a human can review them.

## What it does

When a PR is opened or reopened:

1. Counts how many PRs the same author currently has open (drafts included).
2. Optionally skips the check for trusted authors (`OWNER` / `MEMBER` / `COLLABORATOR`) and for an allowlist of bot accounts.
3. If the count still exceeds the configured limit, the action:
   - Posts a comment on the new PR explaining why it's being closed (only once, even on repeated reopen).
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
          max-open-prs: 5
```

The workflow uses `pull_request_target`, so it must exist on the default branch to take effect. Changes inside a PR do not run until merged.

## Inputs

| Name                   | Required | Default               | Description                                                                                       |
|------------------------|----------|-----------------------|---------------------------------------------------------------------------------------------------|
| `max-open-prs`         | no       | `5`                   | Maximum number of open PRs allowed per author. New PRs that exceed are closed.                    |
| `bot-allowlist`        | no       | `''`                  | Comma-separated bot logins to skip (e.g. `renovate[bot],dependabot[bot]`). Must end in `[bot]`.   |
| `skip-trusted-authors` | no       | `'true'`              | When `'true'`, skip PRs from `OWNER` / `MEMBER` / `COLLABORATOR` authors.                         |
| `github-token`         | no       | `${{ github.token }}` | Token used to list and close PRs. Needs `pull-requests: write`.                                   |

## Configurable inputs via repository variables

To change inputs without editing the workflow, wire them to repo variables:

```yaml
- uses: mirkoalicastro/pr-author-limit@v1
  with:
    max-open-prs: ${{ vars.MAX_OPEN_PRS_PER_AUTHOR || '5' }}
    bot-allowlist: ${{ vars.BOT_ALLOWLIST || '' }}
    skip-trusted-authors: ${{ vars.SKIP_TRUSTED_AUTHORS || 'true' }}
```

Create the variables under **Settings → Secrets and variables → Actions → Variables → New repository variable**. No workflow restart is needed; the next PR opened picks up the new values.

## Permissions

The workflow needs `pull-requests: write` on the `GITHUB_TOKEN`. No PAT or secret is required.

`pull_request_target` runs in the context of the base branch, so the token has write access even for PRs from forks. The action does not check out PR code, so there is no risk of executing untrusted code from a fork.

## Behavior notes

- Triggers: `opened` and `reopened` PRs only. Existing PRs are not affected retroactively.
- The author count includes the newly opened PR itself and includes drafts.
- Counting uses the GitHub search API (`is:pr is:open author:<login>`)
- The comment marker `<!-- pr-author-limit:auto-close -->` prevents duplicate comments on repeated reopen events.
- Trusted authors (`OWNER` / `MEMBER` / `COLLABORATOR`) are skipped by default. Set `skip-trusted-authors: 'false'` to enforce the limit on them too.
- Bot accounts are subject to the limit unless listed in `bot-allowlist`. Logins must include the `[bot]` suffix (e.g. `renovate[bot]`).

## License

See [LICENSE](LICENSE).
