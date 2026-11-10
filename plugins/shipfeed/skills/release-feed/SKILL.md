---
name: release-feed
description: Build a human-readable release feed from a repo's commit history and link commits to changelog entries with the shipfeed CLI. Use when asked for release notes, a ship log, changelog coverage, or why a commit was classified or linked a certain way.
---

# Release feed with shipfeed

`@three-ws/shipfeed` is a zero-dependency Node (>=18) package and CLI. It parses commits (Conventional Commits 1.0.0), classifies each by audience, and links changelog entries to the commits that produced them. Run it with `npx`, no install needed.

## Commands

```bash
npx @three-ws/shipfeed                       # current repo (from the git origin remote), read from GitHub
npx @three-ws/shipfeed --local               # current repo, from local git log
npx @three-ws/shipfeed --repo owner/name --limit 300
npx @three-ws/shipfeed --changelog <url|path>   # link commits against a changelog JSON
npx @three-ws/shipfeed explain <sha>         # why one commit was classified and linked as it was
npx @three-ws/shipfeed stats                 # velocity, audience mix, changelog coverage
npx @three-ws/shipfeed post --telegram --chat <id>   # dry run; add --send to deliver
```

Options: `--branch <name>` (default main), `--limit <n>` (default 200), `--site <url>` for permalinks, `--scopes a,b,c` for scopes that mean "product", `--min-audience holder|developer|internal`, and the output switches `--json`, `--markdown`, `--rss`, `--no-color`.

## Environment

- `GITHUB_TOKEN` or `GH_TOKEN`: optional, raises GitHub's unauthenticated limit of 60 requests an hour.
- `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`: only for `post --send`. Never send without the user's go-ahead, post mode is a dry run until `--send`.

## Library use

```js
import { shipfeed, classify, linkCommits, renderMarkdown } from '@three-ws/shipfeed';
const feed = await shipfeed({ repo: 'owner/name', changelogUrl: 'https://example.com/changelog.json', limit: 200 });
console.log(renderMarkdown(feed));
```

Other exports: `buildShipFeed`, `parseCommitMessage`, `headline`, `filterByAudience`, `groupIntoShips`, `renderRss`, `renderTerminal`, `fetchGitHubCommits`, `fetchChangelog`, `commitsFromGitLog`. The feed has `releases` (each with `commits`, `stats`), `ships` (bursts of work never written up) and `stats.coverage`.

## Worked example

To draft release notes for the last 100 commits of the current checkout:

1. `npx @three-ws/shipfeed --local --limit 100 --markdown > RELEASES.md`
2. `npx @three-ws/shipfeed stats` to see how much of the work has no changelog entry.
3. For a surprising link, `npx @three-ws/shipfeed explain <sha>` prints the score and reasons.

Merge commits and dependency bumps are flagged `noise: true`. A `Changelog: <slug>` trailer in a commit always wins over the heuristic link.
