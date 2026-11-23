# FAQ

**Why don't my numbers ever decrease?**
Because that's the point. The traffic API reports *cumulative* totals, and Odometer additionally keeps the maximum it has ever seen per day — a bad release day can't make your README look sad. Want day-over-day deltas? They're one query away in `stats/daily.csv`; the data is plain.

**Which token do I need?**
One fine-grained personal access token with **Administration: Read only** on the target repo (Setup, step 1). The workflow only ever issues the four `GET`s listed in Setup; every write to your repo uses GitHub's per-run token, which exists for that run and then vanishes.

**Does it work without a token?**
Partly. Stars, forks, release downloads and npm need none — without `TRAFFIC_TOKEN` those still update, one warning is logged, and the views/clones columns stay empty until the secret exists.

**Does it work for a private repo?**
Yes, for any repo the token has access to. Nothing leaves your repo except the four API reads; the block lands in your own README.

**What happens when the token expires or gets revoked?**
The next run logs a clear warning, commits everything it still can, and the traffic columns simply stay empty for the days the token was dead. Paste the new token under the same `TRAFFIC_TOKEN` name; the next run picks it up.

**Can I run it more than once a day?**
Change the cron. Each run merges the maximum-of-day into today's row, so extra runs just make the "last 14 days" table more current.

**What if I rename the repo?**
Nothing breaks — owner/repo are recomputed from `github.repository` on every run and the history continues.

**Why does the Odometer repo show two workflow files in Actions?**
`odometer.yml` is the engine other repos call; `refresh.yml` is the small timer that runs this very repo on itself so its stats are real from day one. A normal install creates exactly one file of its own: `refresh.yml`.

**Rate limits?**
Four reads per run; daily is nowhere near any limit. A very aggressive cron could hit read caps — that run fails loudly, the next one is fine.

**I have a monorepo with several npm packages.**
One package per install — pick the one people actually install. npm reports per-package, so the other numbers never appear here anyway.

**Can I hide the block and keep the CSV?**
The block re-inserts itself on the next run — the CSV is the source of truth. Remove the workflow and the files simply stop updating; nothing is ever deleted from your repo by Odometer.

**I want a different shape or badge.**
Fork the engine, point the `uses:` line at your fork, and change the render step. The collect step doesn't care how the numbers are presented.
