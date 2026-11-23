# Odometer

**Odometer** adds one clean stats line to a repo's README and keeps it current, by itself:

> **Total clones:** 4,352 (github: 210 · npm: 4,142) · **Repo views:** 1,289 · **Stars:** 42 · **Forks:** 7 · **Release downloads:** 356

A reusable workflow (Caqtus/odometer) that runs on your schedule, in your repo:

- collects repo views, clones, stars, forks, release downloads, npm downloads
- appends to `stats/daily.csv` — full history, plain text, yours forever
- rewrites **one managed block** between `repo-stats:begin` / `repo-stats:end` markers — nothing else in the README is ever touched
- numbers **never go down**: the traffic API reports cumulative totals, and Odometer keeps the maximum per day

All writing is done with GitHub's own per-run token. The token you add is read-only.

**Quick start → [Setup](Setup)** · **Details → [What you get](What-you-get)** · **Questions → [FAQ](FAQ)**
