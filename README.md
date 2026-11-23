![Odometer](assets/banner.png)

# Odometer

One clean stats line in your README that updates by itself — just like the block in the picture above.

**Clones (GitHub + npm) · views · stars · forks · release downloads**, as cumulative totals that never go down. Each day the workflow appends one row to a CSV in your repo and rewrites the line from real numbers, so it can't lie. Everything it produces is plain text you own — leave GitHub and the history stays yours.

## Install

### 1 · Token

One fine-grained personal access token, read-only:

1. GitHub → avatar (top-right) → **Settings** → **Developer settings** (bottom of the sidebar) → **Personal access tokens** → **Fine-grained tokens** → **Generate new token**
2. **Resource owner** — your account
3. **Expiration** — **No expiration**
4. **Repository access** — **Only select repositories** → the repo whose stats you want
5. **Repository permissions** — **Administration: Read only** (that is all the traffic endpoints need; `Metadata: Read only` is granted automatically)
6. **Generate token** — copy it, it is shown exactly once

The token is **read-only by design**: the workflow only ever `GET`s your repo's meta and releases plus the two traffic endpoints. Every write to your repo uses a token GitHub hands the workflow for that one run, which dies with the run. Revoke the token anytime from the page where you created it.

### 2 · Secret

Your repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**:

- **Name** — `TRAFFIC_TOKEN`
- **Secret** — the token from step 1

### 3 · Workflow file

Create the file `.github/workflows/odometer.yml` in your repo — web UI: above the file list click **`+`** → **Create new file**, type the name exactly, the folders are created on commit — with this content:

```yaml
name: odometer
on:
  schedule:
    - cron: "0 20 * * *"   # 20:00 UTC = 00:00 (midnight) Georgian time (UTC+4); change the hour for another time
  workflow_dispatch: {}    # gives you a "Run workflow" button
jobs:
  refresh:
    permissions:
      contents: write       # lets the run token commit the stats back — do not remove
    uses: Caqtus/odometer/.github/workflows/odometer.yml@v1
    with:
      npm_package: ""      # your npm package name here (e.g. armazi-hud), or "" if there is none
    secrets: inherit
```

- **`permissions: contents: write`** — the engine commits the updated stats and README block back into your repo with its run token (your token never writes). Without this line a read-only repo default rejects the run. Keep it.
- **`npm_package`** — the exact npm package name if this repo publishes one; leave it `""` otherwise. The npm API is never called and every npm part is hidden automatically; a wrong or missing name never breaks a run.
- **`secrets: inherit`** — passes `TRAFFIC_TOKEN` from your repo into the shared workflow. Keep it.

Commit. Then **Actions** tab → your workflow → **Run workflow**: the block appears within minutes and updates itself daily from then on.

## What you get

- the **stats block** in the README — self-healing: if anything removes it (a tool, an AI, you), the next run puts it back
- `stats/README.md` — totals, last-14-days and weekly tables
- `stats/daily.csv` — one row per day, the full history
- `stats/downloads.csv` — per-release download counts, if you have releases

Full detail: the `docs/` folder — Home, Setup, What-you-get, FAQ.

## FAQ

**Why don't the numbers ever decrease?** GitHub's traffic API reports *cumulative* totals, and Odometer additionally keeps the maximum it has ever seen per day. Day-over-day deltas are one query away in `stats/daily.csv`.

**Can I use a fine-grained token?** Yes — that is the recipe above. `Administration: Read only` is exactly what the traffic endpoints require; the workflow never writes with your token in any case.

**Does it work without a token?** Partly. Stars, forks, release downloads and npm need none; views and clones do. Without `TRAFFIC_TOKEN` the run stays green, logs one warning, and the traffic numbers stay empty.

**Does it work for a private repo?** Yes, for any repo the token has access to. The block lands in your own README, not on a third-party service.

**What if the token is revoked or expires?** The run stays green: stars/forks/releases keep updating, a warning is logged, traffic stays empty until you paste a fresh token under the same secret name. No code changes.

**Why does this repo show two workflow files?** `odometer.yml` is the engine every install calls; `refresh.yml` is the small timer that runs this very repo on itself, so the stats above are real from day one. Your install has exactly one file.

**Uninstall?** Delete `odometer.yml`, the `stats/` folder, the block between the `repo-stats:begin` / `repo-stats:end` markers, and the `TRAFFIC_TOKEN` secret. Nothing else.
