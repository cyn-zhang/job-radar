---
description: Draft a follow-up or thank-you email for a job application
---

Run JobRadar follow-up email module. Load config and tracker from config.yaml and applications/tracker.json, then follow Module 12 in SKILL.md.

Usage examples:
- /job-followup — scan for stale applications (14+ days) and offer to draft follow-ups
- /job-followup Atlassian — draft a follow-up for a specific company
- /job-followup thank-you — draft a thank-you email after an interview

Rules:
- Never send emails automatically. Draft only; user reviews and sends manually.
- If application is less than 7 days old, advise waiting before following up.
- After user confirms they sent the email, update next_step in tracker.json.
