# What you get

After N days of running, your repo contains exactly this and nothing else:

```
README.md            ← one managed block (auto-maintained, see below)
stats/
  daily.csv          ← full history: one row per day since day 1
  README.md          ← prettified stats page
  downloads.csv      ← release download counts (only if you have releases)
```

## The README line

One bold line, metrics separated by `·`, zero-valued or unconfigured metrics
**omit themselves** (no "npm: 0" noise):

> **Total clones:** 4,352 (github: 210 · npm: 4,142) · **Repo views:** 1,289 · **Stars:** 42 · **Forks:** 7 · **Release downloads:** 356

- `Total clones` = github clone/fetch **operations** + npm package downloads,
  broken out as `(github: … · npm: …)`
- no npm package? → just `**Total clones:** 210` — npm never appears
- brand-new repo, nothing collected yet? → `**no metrics yet** — collecting`

It links to `stats/README.md` ("Full stats").

## The managed block — the self-healing part

The line lives between two markers:

```
<!-- repo-stats:begin -->
...the line...
<!-- repo-stats:end -->
```

Rules:

- the block is **never** edited by hand — if you delete or change it, the next
  run just re-inserts/updates it (that's the "self-heal")
- the rest of your README is never touched — no reformatting, no reordering
  (the workflow re-inserts at the same anchor position, below the first image
  or above the first heading)
- the block carries an HTML comment telling AI tools to leave it out when
  rewriting the file — the workflow puts it back

## `stats/README.md`

The detailed page:

- **Totals** — every metric with its value and the last-14-days column
- footnotes explaining cumulative vs. per-day semantics
- **Release downloads** — per-tag, per-file table (if you have releases)
- **Daily — last 14 days** — table
- **Weekly** — this week + up to 25 prior weeks, with a note that the current
  week is still in progress

## `stats/daily.csv`

Plain CSV, one row per day:

```
date,views,unique_visitors,clones,unique_cloners[,npm_downloads]
```

(the `npm_downloads` column exists only if you configured an npm package)

Every row is the **maximum of all runs that have ever reported that day** —
if GitHub revises a number upward, your CSV quietly rises to match; it never
goes back down.
