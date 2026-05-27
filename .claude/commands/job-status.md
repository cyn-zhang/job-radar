---
description: Show application tracker dashboard
---
Show a summary of all active job applications using JobRadar. Load config from config.yaml, then follow Module 10 in SKILL.md.

Read `applications/tracker.json` (source of truth) and present:
- A status summary table (grouped: Offers → Interviews → Applied → Watchlist → Needs Attention → Closed)
- Upcoming deadlines flagged 🔴 (within 7 days)
- Stale applications flagged ⚠️ (no status change in 14+ days)
- A one-line recommended next action

Pass "closed" to expand the closed list: /job-status closed
