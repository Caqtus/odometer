![Odometer](assets/banner.png)

# Odometer

One clean stats line in your README that updates by itself and never lies:
**total clones (github + npm) · repo views · stars · forks · release downloads.**

Every day it looks up the public numbers for your repo, adds one row to a CSV
file **in your repo**, and rewrites a single block in the README. The numbers
only ever grow. Everything it produces is plain text you own — fork it, archive
it, or delete GitHub; your history stays yours.

## Install (3 steps, about 5 minutes)

### Step 1 — Create a token (a password the workflow uses to *read* your stats)

1. GitHub → click your **avatar** (top-right) → **Settings**
2. In the left sidebar, scroll to the very bottom → **Developer settings**
3. **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**
4. Fill in the form:
   - **Note** — anything, e.g. `odometer-read`
   - **Expiration** — choose **No expiration** (a permanent token works fine here)
   - **Select scopes** — check **`repo`** and nothing else
5. Click **Generate token**.
   - If you have 2FA on, GitHub asks for your 2FA code right at this moment.
   - The token appears **exactly once** — copy it now, you can't see it again.

A token is just a password that lets the workflow read your public stats
(views, clones, stars…). It is used **read-only** — Odometer never pushes or
writes with it; all writing is done by a throwaway password GitHub hands the
workflow for that one run. A permanent token is safe: it can only read, and
you can delete it anytime (Settings → the same page → *Revoke*).

> Fine-grained tokens can't be used for the views/clones endpoints — use the
> classic token above.

### Step 2 — Add the workflow file

1. Open your repo page (github.com/your-name/your-repo).
2. Look at the file list, just above it: click the **`+`** icon → **Create new file**
   (a smaller menu: if you see "Add file", click that first, then *Create new file*).
3. In the name box, type exactly:
   `.github/workflows/odometer.yml`
   Don't press anything yet — GitHub will create the folders for you when you
   commit.
4. Paste this into the big editor:

   ```yaml
   name: odometer
   on:
     schedule:
       # daily at 20:00 UTC = 00:00 (midnight) Georgian time (UTC+4);
       # change "20" to a different UTC hour if you want another time
       - cron: "0 20 * * *"
     workflow_dispatch: {}    # gives you a "Run workflow" button anytime
   jobs:
     refresh:
       uses: Caqtus/odometer/.github/workflows/odometer.yml@v1
       with:
         # put your npm package name here, e.g.  npm_package: my-package
         # (only if this repo publishes an npm package — otherwise leave it "")
         npm_package: ""
       secrets: inherit
   ```

   - **Leaving `npm_package: ""` is the default and it's safe.** If you have an
     npm package, replace `""` with its name (e.g. `armazi-hud`). If you leave
     the value wrong or empty, the run **doesn't break** — the npm part is just
     hidden.
   - `secrets: inherit` (last line) is what passes your token from Step 1 to
     the workflow — keep it.
5. Scroll down → click the blue **Commit changes** button → on the small page
   that follows, click **Commit new file**.

### Step 3 — Store your token as a secret

1. Your repo → **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret** (top of the page)
3. **Name**: `TRAFFIC_TOKEN` (exact spelling)
   **Secret**: paste the token from Step 1
4. Click **Add secret**.

### First run

Open the **Actions** tab at the top of your repo → click your workflow's name
(`odometer`) in the left list → **Run workflow** button (top-right). Within a
couple of minutes your README gets its first stats block, and from then on it
updates itself every day at the time you set.

## What you get

- a one-line **stats block** in the README — self-healing: if anything (including
  an AI tool) rewrites the README and drops the block, the next run puts it back
- `stats/README.md` — totals, last-14-days table, weekly rollups, footnotes
- `stats/daily.csv` — the full history, one row per day
- `stats/downloads.csv` — release download counts (only if you have releases)

More detail: see the **wiki** tab (top-right of the repo page).

## Questions

**Do I have to create or manage a token?** Odometer itself never creates
anything in your account. The only password you make is the read-only
`TRAFFIC_TOKEN` from Step 1. Everything the workflow *writes* is done with
GitHub's own per-run password, which dies when the run ends.

**Why are there two workflow files in *this* repo's Actions?** One is the kit
itself (`odometer.yml` = the engine others call), the other (`refresh.yml`) is
the small timer that runs this very repo on itself, so the stats in this README
are real from day one. A normal install only has **one** file (Step 2 above).

**Uninstalling?** Delete the workflow file, delete the `stats/` folder, remove
the block between the `repo-stats:begin` / `repo-stats:end` markers in the
README, and delete `TRAFFIC_TOKEN` in Settings. Nothing else to clean up.
