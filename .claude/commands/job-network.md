---
description: Alumni networking — map where alumni work and draft personalised outreach
---
Run JobRadar alumni networking. Load config from config.yaml, then follow Module 11 in SKILL.md.

Usage examples:
- /job-network --spike — validate whether LinkedIn alumni search is extractable before building automation
- /job-network --map — build alumni company distribution and save networking/alumni_map.json
- /job-network --list — show saved alumni grouped by company
- /job-network --list Atlassian — show saved alumni at one company
- /job-network --reach Atlassian — generate personalised outreach drafts for alumni at Atlassian
- /job-network --prep "Sarah Chen" — prepare coffee chat questions

Tracking:
- Alumni map lives in networking/alumni_map.json
- Long-term outreach status lives in applications/tracker.json under networking[]
- When the user drafts, sends, gets a reply, schedules a chat, or receives a referral, update both stores

Safety rules:
- Never send LinkedIn messages automatically.
- User must review and manually send every message.
- Store alumni data only in networking/ (gitignored).
