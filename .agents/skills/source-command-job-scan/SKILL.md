---
name: "source-command-job-scan"
description: "Scan all job boards and save results to scans/"
---

# source-command-job-scan

Use this skill when the user asks to run the migrated source command `job-scan`.

## Command Template

Run a daily job scan using JobRadar. Load config from config.yaml, then follow Module 1 in SKILL.md. Search all enabled sources, score and rank results, present the table sorted by App Closes, and save to scans/Jobs_{YYYY-MM-DD}.md.
