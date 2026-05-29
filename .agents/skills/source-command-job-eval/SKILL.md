---
name: "source-command-job-eval"
description: "Evaluate a JD — requirements, ATS keywords, fit verdict"
---

# source-command-job-eval

Use this skill when the user asks to run the migrated source command `job-eval`.

## Command Template

Evaluate a job description using JobRadar. Load config from config.yaml, then follow Module 2 in SKILL.md. Save the raw JD immediately, extract requirements and ATS keywords, produce a fit summary, save jd_analysis.md, then offer to run the coverage map (Module 5).

If no JD was provided with this command, ask: "Please paste the job description or drop a link."
