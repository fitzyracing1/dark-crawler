# ○ dark-crawler

**Scan popular GitHub repos for dark CI lights.**

A dark light is any completed push/schedule workflow run on the default branch (last 30 days) whose conclusion is not `success`. **lit** = success = bit `0`. **dark** = anything else = bit `1`.

The live scheduled instance runs from [fitzyracing1/darks](https://github.com/fitzyracing1/darks). Board: **https://light-report-production.up.railway.app/darks**

Extracted from [fitzyracing1/light-report](https://github.com/fitzyracing1/light-report) (`bin/dark-crawl.mjs` + `lib/darks.mjs`).

## What it does

1. Collects candidate repos (most-starred, trending, recently active popular, MCP servers, top npm package repos) plus any open darks and optional seed files.
2. For each repo, applies the **strict rule** per workflow that still exists on the default branch:
   - latest completed **push** or **schedule** run on the default branch within **30 days**
   - `success` → **lit**; anything else (failure, cancelled, timed out, …) → **dark**
   - newer lit run of the same workflow wins
   - ignores `pages-build-deployment`, Dependabot, dynamic CodeQL
3. For each dark light, picks a one-line **cause** from check-run annotations and a **class** (`code bug` · `flaky` · `infra` · `external`).
4. Merges into `darks.json`, refreshes `QUEUE.md`, and (unless disabled) syncs **one issue per dark** in `DARKS_REPO` only.

### Hard rule

**The crawler never opens issues, PRs, or comments on anyone else's repo.** Every write goes through `ownRepoWrite()`, which refuses any path outside `DARKS_REPO` (default `fitzyracing1/darks`). Proposed upstream actions only go into `QUEUE.md` / `queued.json` for approval; nothing is sent from here.

## Run locally

Needs Node 18+. No installable dependencies.

```bash
git clone https://github.com/fitzyracing1/dark-crawler.git
cd dark-crawler

# help
node bin/dark-crawl.mjs --help

# dry run: scan but write nothing and open no issues
GITHUB_TOKEN=ghp_... node bin/dark-crawl.mjs --dir /tmp/darks-out --max-repos 5 --dry-run

# full local run into a state directory (writes darks.json / QUEUE.md; issues only in DARKS_REPO)
mkdir -p /tmp/darks-out
GITHUB_TOKEN=ghp_... DARKS_REPO=fitzyracing1/darks \
  node bin/dark-crawl.mjs --dir /tmp/darks-out --max-repos 20 --no-issues
```

Or via the bin name after linking:

```bash
npm link   # exposes `dark-crawl`
dark-crawl --help
```

## Options

| option | meaning |
|---|---|
| `--dir <path>` | directory for `darks.json`, `QUEUE.md`, `status.json`, `queued.json` (default `.`) |
| `--max-repos <n>` | max repos to scan this run (default `120`) |
| `--no-issues` | do not create/update/close issues in `DARKS_REPO` |
| `--seed <file>` | seed repos from a `.tsv` / `.json` / text file (repeatable) |
| `--only <owner/repo>` | scan only these repos (repeatable; skips candidate search) |
| `--sync-only` | re-sync issues from existing `darks.json`; no scan |
| `--dry-run` | scan (or sync) but write nothing and open no issues |
| `-h`, `--help` | show help |

## Environment variables

| env | meaning |
|---|---|
| `GITHUB_TOKEN` | GitHub token used for API reads (and writes to `DARKS_REPO` when issues are enabled) |
| `DARKS_REPO` | only repo the crawler may write to (default `fitzyracing1/darks`) |

## Scheduled instance

The production schedule lives in [fitzyracing1/darks](https://github.com/fitzyracing1/darks) (every 6 hours). See [`examples/crawl.yml`](examples/crawl.yml) for a copy of that workflow adapted to this package. That example is **not** under `.github/workflows/` here, so it does not run and creates no issues in this repo.

Live board: https://light-report-production.up.railway.app/darks

## Tests

```bash
npm test   # node --check on the CLI/lib, then node --test
```

## License

MIT © 2026 Joshua Almeida

## Plans

This project is free, and its license stays as is. Support it through [GitHub Sponsors](https://github.com/sponsors/fitzyracing1):

- **Free**: use the code, community support.
- **Pro ($49/month)**: priority issues and support.
- **Team ($199/month)**: priority support for your team plus early access.
