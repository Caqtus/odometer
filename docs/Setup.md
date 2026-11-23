# Setup

Three steps. ~5 minutes. (This is the same way you set up the repo that hosts Odometer — it dogfoods itself.)

## Step 1 — Create a personal access token

1. GitHub → your avatar (top-right) → **Settings**
2. left sidebar, very bottom → **Developer settings**
3. **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**
4. Fill in the form:
   - **Note** — anything, e.g. `odometer-read`
    - **Expiration** — choose **No expiration**. A permanent token works fine: it is used read-only, and you can revoke it anytime from this same page
   - **Select scopes** — check **`repo`** and nothing else
5. Click **Generate token**.
   - **2FA?** If you have two-factor authentication on, GitHub will ask for your
     2FA code (app or SMS) *right at this moment*, before the token is shown.
   - The token is shown **exactly once**, on the next screen. Copy it now — you can't see it again.

Why a classic token, and why `repo` scope:
the two endpoints that power this (repo **views** and **clones** — the
"traffic" API) accept **classic tokens only**; fine-grained tokens are not a
supported path there. `repo` scope is what GitHub documents for that API.
In practice the token is used **read-only** — the workflow never pushes with it.
Every write to your repo is done by GitHub's own per-run `GITHUB_TOKEN`, which
vanishes when the run ends.

## Step 2 — Store it as a secret

1. Your repo → **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret**
   - Name: `TRAFFIC_TOKEN`
   - Value: paste the token
3. Done. The secret is encrypted at rest on GitHub's side.

## Step 3 — Add the workflow

Create the file `.github/workflows/odometer.yml` in your repo:

1. Repo page → above the file list, click **`+`** → **Create new file**
   (if you see *Add file* instead, click it, then *Create new file*)
2. Name box: `.github/workflows/odometer.yml` — GitHub creates the folders for you
3. Paste this into the editor:

```yaml
name: odometer
on:
  schedule:
    # daily at 20:00 UTC = 00:00 (midnight) Georgian time (UTC+4); change "20"
    # to another UTC hour if you want a different time
    - cron: "0 20 * * *"
  workflow_dispatch: {}    # gives you a "Run workflow" button anytime
jobs:
  refresh:
    uses: Caqtus/odometer/.github/workflows/odometer.yml@v1
    with:
      # your npm package name here, or "" if there is no npm package (safe default)
      npm_package: ""
    secrets: inherit
```

4. Blue **Commit changes** button at the bottom → **Commit new file**

- the reference points at the public Odometer repo (`Caqtus/odometer`)
- `npm_package` — put the exact npm package name (e.g. `armazi-hud`) **only if**
  this repo publishes one. Leave it `""` (the default above) if it doesn’t — the
  npm API is never called and all npm parts are hidden automatically. A wrong or
  missing name never breaks a run.
- `secrets: inherit` passes `TRAFFIC_TOKEN` from your repo into the called workflow.

Then open **Actions → odometer → Run workflow** for the first run (or just wait
for the cron). The first run is quiet on purpose: most counters are still 0, so
the line will be short — it grows as real days accumulate.

## Verify

In your repo you should now see, besides the README line:

- `stats/daily.csv` — one row per day
- `stats/README.md` — totals table, last-14-days table, weekly rollups
- `stats/downloads.csv` — if you have releases with assets

All of it is plain text in **your** repo. Fork it, archive it, delete GitHub —
the history stays yours.
