---
name: "source-command-job-track"
description: "Add or update an application in the tracker"
---

# source-command-job-track

Use this skill when the user asks to run the migrated source command `job-track`.

## Command Template

Update the application tracker using JobRadar. Load config from config.yaml, then follow Module 9 in SKILL.md.

Source of truth is `applications/tracker.json`. `applications/tracker.md` is auto-regenerated — never edit it directly.

If no arguments provided, show the current tracker table (read from tracker.json) and ask what to update.

Usage examples:
- /job-track — show current tracker
- /job-track applied Atlassian "Software Engineer Intern" — add new application
- /job-track watchlist Canva "Product Designer Intern" — save to watchlist before applying
- /job-track interview NAB "Technology Graduate" — update status
- /job-track offer Canva "Product Engineer Intern" — update status
- /job-track rejected Google "Software Engineer Intern" — update status

Status values: watchlist | applied | oa | interview | final_round | offer | accepted | rejected | withdrawn | ghosted
