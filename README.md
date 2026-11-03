# DevQuay sync action

Keeps the DevQuay files in a repository (`AGENTS.md`, `CLAUDE.md`, `.cursor/rules/devquay-*.mdc` and `.devquay/`) in step with the documents its DevQuay project adopts. It runs [`npx devquay sync`](https://www.npmjs.com/package/devquay) and opens or updates a pull request when something changed. It never pushes to your default branch.

## Use

```yaml
name: DevQuay sync
on:
  schedule:
    - cron: "0 6 * * 1-5"
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: devquay/sync-action@v1
        with:
          api-key: ${{ secrets.DEVQUAY_API_KEY }}
          project: payments-service
          format: agents-md,cursor
```

To fail CI instead when the files are out of date, use `mode: check` in your pull request workflow; it needs no write permissions.

## Inputs

| Input | Default | |
|---|---|---|
| `api-key` | (required) | A workspace API key from DevQuay › Settings › API keys. Read scope is enough. Store it as a secret. |
| `project` | the project listing this repository | The DevQuay project slug. |
| `format` | `agents-md` | `agents-md`, `claude-md`, `cursor` or `all`, comma-separated for several. |
| `mode` | `pull-request` | `pull-request`, `check` (fail when out of date) or `write` (only write the files, for your own later steps). |
| `api-url` | `https://api.devquay.com` | The DevQuay API. |
| `cli-version` | `0` | The `devquay` CLI version for `npx`. |
| `branch` | `devquay/sync` | The pull request branch. |
| `token` | `github.token` | Opens the pull request. Use a PAT or app token if your workflows must run on it. |

## Outputs

| Output | |
|---|---|
| `changed` | `true` when any DevQuay file changed. |
| `pull-request-url` | The pull request, in `pull-request` mode. |

The action only touches DevQuay's own files and compares no code: it is not drift detection. Text you write outside the `<!-- devquay:start -->` markers in `AGENTS.md` and `CLAUDE.md` is kept.
