# Odometer

**Odometer** adds one clean stats line to your repo's README and keeps it current,
by itself:

> **Total clones:** 4,352 (github: 210 · npm: 4,142) · **Repo views:** 1,289 · **Stars:** 42 · **Forks:** 7 · **Release downloads:** 356

It is a reusable GitHub Action (`Caqtus/odometer`):

- runs on **your** schedule (default daily), on **your** repo
- collects repo views, clones, stars, forks, release downloads, npm downloads
- appends to `stats/daily.csv` in **your** repo (full history, yours forever)
- rewrites **one managed block** between `repo-stats:begin` / `repo-stats:end` markers — nothing else in your README is ever touched
- numbers **never go down**: GitHub's traffic API reports cumulative totals, and Odometer keeps the maximum per day

Because the workflow does all writing with its own throwaway GitHub token, the token
**you** add is only ever used to *read* the traffic API.

**Quick start → [Setup](Setup)** · **Details → [What you get](What-you-get)** ·
**Questions → [FAQ](FAQ)**
