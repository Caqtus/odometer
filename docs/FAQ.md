# FAQ

**Why don't my numbers ever decrease?**
Because that's the point. GitHub's traffic API reports *cumulative* counts
(since the API started tracking the repo), not "since last run". So each day's
row is a floor, and Odometer keeps the maximum it has ever seen per day. A day
where you shipped a bad version still doesn't make your README look sad.
If you *want* day-over-day deltas, they're trivial to compute from
`stats/daily.csv` — the data is plain.

**Why does the token need `repo` scope? Isn't that a lot?**
The scope list for classic tokens is coarse. The two endpoints that power
Odometer (views + clones, the "traffic" API) require a token with `repo`
(scoping to read-only is possible on the API side but GitHub doesn't expose a
`traffic:read` classic scope). In practice the workflow only ever calls
`GET /repos/{owner}/{repo}/traffic/views`, `GET /repos/{owner}/{repo}/traffic/clones`,
`GET /repos/{owner}/{repo}` and `GET /repos/{owner}/{repo}/releases`.
It never does a `POST`, `PUT`, `DELETE`. All writes to your repo are done
with the GitHub-hosted `GITHUB_TOKEN` that exists for one run and then
disappears.

**Can I use a fine-grained token instead?**
Not for the traffic endpoints — fine-grained tokens do not expose
`/traffic/*`. If you want Odometer to work at all, use a classic one.
(If you only care about stars/forks/release counts, you could skip the
token entirely — the workflow falls back to anonymous and keeps working —
but you lose views and clones.)

**Does it need a server, a database, or a dashboard?**
No. There is no state anywhere but the CSV on your default branch.
Fork, migrate, self-host — the file is the state.

**What happens if I rename my repo?**
Nothing breaks: the workflow computes the owner/repo from
`github.repository` every run. Historical rows stay, new ones continue.

**What happens if the token expires?**
The next run logs a clear warning
(`TRAFFIC_TOKEN secret expired — traffic metrics paused; stars/forks/releases still work`),
keeps committing everything it still can (stars, forks, release downloads, npm),
and your CSV just doesn't get its traffic rows filled in for the days the
token was dead. Regenerate, paste the new one under the same secret name,
and the next run picks it up. No code changes.

**Can I run it twice a day?**
Sure — change the cron. Each run appends/updates today's row with the
maximum value seen so far that day, so running more often just fills in the
"last 14 days" table a little bit earlier.

**Why does the Odometer repo itself show two workflow files in Actions?**
One is the kit's engine (`odometer.yml` — the reusable workflow other repos
call), the other (`refresh.yml`) is a tiny timer that runs this very repo on
itself, so the stats in this repo's README are real from day one.
A normal install creates only **one** file (Setup, step 3).

**What about rate limits?**
The traffic API is one call per run per endpoint. Daily runs are nowhere
near the limit. If you schedule very aggressively (every minute, say) you'll
hit the unauthenticated-style read caps fast; the workflow fails loudly on
`HTTP 403` and the next day's run is fine.

**Does it work for private repos?**
Yes. The traffic API is available for any repo the token can read, and
`TRAFFIC_TOKEN` with `repo` scope can read private repos you're a member of.
Your stats block ends up in your private README, not on some third-party
service.

**I have a monorepo with multiple npm packages.**
Odometer reads one package name per run. For a monorepo, run Odometer once
for the primary package — the other packages' numbers aren't in the GitHub
traffic API anyway (npm reports per-package, so you'd pick the one people
actually install).

**Can I delete the stats block and keep the CSV?**
Sure. The block re-inserts itself on the next run but the CSV is the source
of truth — delete the block any time you want a clean look, re-add it by
re-running. Conversely, delete the workflow and the files stop updating;
they don't get removed.

**I want a different badge / a different layout.**
Odometer is deliberately one format: the statline. If you want a different
shape, fork the reusable workflow, point your `.github/workflows/odometer.yml`
at your fork with `uses: your-login/odometer/...`, and change the render
step. The collect step is agnostic to how you present the numbers.
