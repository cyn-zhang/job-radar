# JobRadar Skill

## Setup
Copy config.example.yaml to config.yaml and fill in your personal details before first use.

## Paths
base_path: applications/
skill:     SKILL.md
# cv_path and all other user settings live in config.yaml — edit that file, not here.

## File Structure
All job files saved under:
applications/{Company}/{Job_Title}/
  ├── jd.md
  ├── jd_analysis.md
  ├── coverage_map.md
  ├── gaps_and_improvements.md
  ├── recruiter_review_and_suggestions.md
  ├── CV_{Company}_{Role}_{date}.docx
  └── CoverLetter_{Company}_{Role}_{date}.docx

Daily scan saved to:
scans/Jobs_{YYYY-MM-DD}.md

## Sources
- Seek: seek.com.au
- LinkedIn: linkedin.com/jobs
- GradConnection: au.gradconnection.com
- Aus Internship Finder: aus-internship-finder.vercel.app (load references/aus-internship-finder.md — do NOT use site: search)
- Indeed: indeed.com.au
- Company career pages: see config.yaml target_companies

## Browsing
If gstack is installed (`.claude/skills/gstack/`), use `/browse` for all web browsing — it handles JavaScript-rendered pages and sites that block WebFetch (LinkedIn, some company career pages).
If gstack is not available, fall back to WebSearch + WebFetch. Flag any sources that return 403 or no results in the scanner notes.

## Models
- Job scan, JD analysis, gap analysis, digest → claude-haiku-4-5-20251001
- CV writing, cover letter → claude-sonnet-4-6

## Gmail
MCP: https://gmailmcp.googleapis.com/mcp/v1

## Cron
Daily digest at 9am:
0 9 * * * cd /path/to/job-radar && claude -p "run daily job scan and send Gmail digest" --model claude-haiku-4-5-20251001

## Behaviour
- Save jd.md immediately on any JD paste — before analysis
- Bottom Line table mandatory on every JD evaluation
- Sort results by App Closes ascending (🔴 first, ⚠️ next, 🟢 last)
- Flag 🔴 anything closing within 7 days
- Always offer next step after each module
- Never pad or fabricate experience in CV or cover letter
