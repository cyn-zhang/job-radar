---
description: Online assessment prep — detect OA type from JD and generate targeted practice plan
---

Run JobRadar OA preparation module. Load config from config.yaml, then follow Module 13 in SKILL.md.

Usage examples:
- /job-oa — scan tracker for applications in OA stage and offer prep for each
- /job-oa Atlassian — prep for a specific company's OA
- /job-oa coding — jump straight to coding assessment prep
- /job-oa video — jump straight to video interview prep
- /job-oa psychometric — jump straight to aptitude test prep

How it works:
1. Reads the JD (from applications/ folder or pasted) to detect OA type
2. Generates a targeted prep plan with resources and tips
3. If deadline is within 7 days, flags urgency
4. Offers to update tracker status to OA in progress
