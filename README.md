# sports-config: `qa` branch

The league configuration the QA-tests suite adds **on top of `main`** on a test environment.
It is a standalone branch: it does not contain `main` and never needs to be rebased on it.

## How it is applied

The prepare step of the regression syncs `main` first, as the admin screen does, and then this
branch with:

```
POST /coil-web/api/leagues/configuration/qa/sync?deleteOrphans=false&createMissingLeagues=false&createChallenges=false
```

`deleteOrphans=false` makes the sync additive (core PR #551): `main`'s leagues keep their
visibility, and this branch only adds its own. Challenges for the QA seasons are created per
season by the tests that need them.

## What is in it

| File | League | Purpose |
|---|---|---|
| `sports/motorsport/world/qaFormula1Payout.yml` | `QA_FORMULA1_PAYOUT`, season `QA_FORMULA1_25_PAYOUT` | A copy of Formula 1 2025 for the Pick5 payout test. Past season: the simulator replays its real results, so the expected winners and payouts are known. Shares the real Formula 1 teams, so their markets and keys. |
| `sports/football/qa/teams.yml`, `qaWhitelist.yml` | `QA_WHITELIST`, season `QA_WHITELIST_26_27` | Four QA-only teams, two whitelisted and two open, with their own markets, for the whitelist permutations. |

## Rules for adding a league here

- **One test, one league.** A test that needs its own challenge round or pool gets its own
  league file. Two seasons of the same real season cannot live in one league (one active
  season per league, and the season key is the real-season id for most scrapers).
- **Copy a past season** to replay real results: same season key (`'2025'`, `'2025-2026'`),
  same scraper names and source ids (`tsdbId`, `espnId`, `goalServeId`), same dates, same
  team ids. New league `code`, new season `code` (unique, at most 45 characters; reusing an
  existing season code moves that season to the new league). `active: true`, `visible: true`.
- **Team ids reference `main`'s teams** unless the league needs teams of its own. Copies share
  the real team's address, market, keys, stakes and vault. QA-only teams go in a `teams.yml`
  of their own folder with a fixed `address`.
- Every file under `sports/<sport>/<country>/` other than `teams.yml` is parsed as a league,
  so put nothing else there.
- Prefix league and season codes and QA team ids with `QA_` / `FOOTBALL_QA_`, so they can be
  told apart on an environment.
