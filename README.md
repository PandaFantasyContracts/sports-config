# sports-config

The league configuration the backend (core) loads: one file per league under
`sports/<sport>/<country>/`, next to that country's `teams.yml`. An environment picks it up when
it is synced, from the admin League configuration screen ("Reload configuration") or with:

```
POST /coil-web/api/leagues/configuration/{branch}/sync
```

The `qa` branch holds the QA-only leagues the regression suite syncs on top of `main`; see that
branch's README.

## Rules for editing

- **The sync never deletes.** It creates and updates leagues, seasons and teams, and leaves alone
  whatever the branch doesn't name: leagues, seasons, teams, pools, challenges and payouts all stay.
  To retire a league, keep its file and set `'visible': !!bool 'false'`.
- **One active season per league.** The sync refuses a league with zero or several active seasons.
  An active season that a change drops or renames is deactivated by the next sync (its row stays),
  so at season rollover prefer keeping the old season with `active` false over deleting it.
- **`challenges` is opt-in.** A sync with `createChallenges=true` creates the active season's
  challenges only for leagues with `'challenges': !!bool 'true'`. Leave it out and it is `false`;
  set it explicitly on every new league.
- **A new key needs its backend first.** The backend rejects a key it doesn't know, or a value of
  the wrong type, and the whole sync fails. Deploy the core version that reads a new or changed key
  to every environment that syncs this branch before merging the change here.
