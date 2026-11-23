# Setup

Three steps, ~5 minutes — the same setup this repo uses on itself.

## 1 · Fine-grained token (read-only)

1. Avatar → **Settings** (top-right) → **Developer settings** (bottom of the sidebar) → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**
2. **Resource owner** — your account
3. **Expiration** — **No expiration**. Fine-grained tokens can be permanent, and here that is safe: it only reads, and one click on this page revokes it.
4. **Repository access** — **Only select repositories** → the repo whose stats you want
5. **Repository permissions** — set **Administration** to **Read only**. Nothing else is needed; `Metadata: Read only` is granted automatically.
6. **Generate token** → copy it (shown once). With 2FA on, GitHub asks for the code at this step.

That is the whole permission surface. The endpoints Odometer calls:

```
GET /repos/{o}/{r}
GET /repos/{o}/{r}/releases
GET /repos/{o}/{r}/traffic/views?per=day
GET /repos/{o}/{r}/traffic/clones?per=day
```

GitHub reports the permission they accept as `administration=read`, which is the one you ticked above. No classic token is needed — fine-grained works.

## 2 · Secret

Repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**:

- **Name** — `TRAFFIC_TOKEN`
- **Secret** — the token

GitHub encrypts it at rest; no one in Settings can read the value back.

## 3 · Workflow file

Create `.github/workflows/odometer.yml` (web UI: **`+`** → **Create new file**, type the name exactly):

```yaml
name: odometer
on:
  schedule:
    - cron: "0 20 * * *"   # 20:00 UTC = 00:00 (midnight) Georgian time (UTC+4); change the hour for another time
  workflow_dispatch: {}    # gives you a "Run workflow" button
jobs:
  refresh:
    uses: Caqtus/odometer/.github/workflows/odometer.yml@v1
    with:
      npm_package: ""      # your npm package name here (e.g. armazi-hud), or "" if there is none
    secrets: inherit
```

- **`npm_package`** — the exact npm package name if this repo publishes to npm (e.g. `armazi-hud`); leave `""` otherwise. A wrong or missing value never breaks a run — the npm parts are hidden automatically.
- **`secrets: inherit`** — forwards `TRAFFIC_TOKEN` into the engine.

Commit, then **Actions → your workflow → Run workflow** for the first run (or just wait for the cron). The first line is short on purpose: counters start at 0 and grow with real days.

## Verify

In your repo you should now see:

- `stats/daily.csv` — a row for today
- `stats/README.md` — totals, last-14-days table, weekly rollups
- the README block between `repo-stats:begin` / `repo-stats:end`

All of it is plain text in **your** repo — fork it, archive it, delete GitHub, and the history stays yours.
