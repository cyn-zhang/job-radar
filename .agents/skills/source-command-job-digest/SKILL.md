---
name: "source-command-job-digest"
description: "Scan jobs + send Gmail digest to inbox"
---

# source-command-job-digest

Use this skill when the user asks to run the migrated source command `job-digest`.

## Command Template

Run the JobRadar daily digest. Load config from config.yaml, then follow Module 7 in SKILL.md. Run the job scan (Module 1), compose the digest email, save the scan to scans/Jobs_{YYYY-MM-DD}.md, then create a Gmail draft via MCP and send to the configured email address.
