---
name: job-radar
description: >
  A fully configurable job hunting assistant. Use this skill for anything related to finding,
  applying for, or preparing for jobs at any level (intern through executive / C-suite) in any
  industry. Triggers include: "find jobs", "search roles", "scan today", "evaluate this JD",
  "analyse this job", "tailor my CV", "customise my resume", "write a cover letter", "gap
  analysis", "am I a good fit", "interview prep", "mock interview", "coding assessment",
  "daily digest", "send me jobs", or any mention of job hunting, job applications, or career
  opportunities. Also trigger when the user pastes a job description or job ad. Configure via config.yaml — no changes to this file needed.
---

# JobRadar

A configurable job hunting assistant. All user preferences are read from `config.yaml`.
No hardcoded values in this file — everything is a variable.

---

## Step 0 — Always Load Config First

Locate `config.yaml` by checking these paths in order — use the first one found:

1. `./config.yaml` — current working directory (project-local config takes priority)
2. `~/.claude/skills/job-radar/config.yaml` — global fallback

Extract:

```
{name}              ← hunter.name
{email}             ← hunter.email
{level}             ← hunter.level
{roles}             ← hunter.roles (list)
{eligible_majors}   ← hunter.eligible_majors (list, may be empty)
{locations}         ← hunter.locations (list)
{industry}          ← hunter.industry
{work_type}         ← hunter.work_type (internship | graduate | contract | permanent | any)
{sources}           ← hunter.sources (map of enabled/disabled)
{target_companies}  ← hunter.target_companies (list, may be empty)
{exclude_companies} ← hunter.exclude_companies (list, may be empty)
{base_path}         ← hunter.base_path
{cv_path}           ← hunter.cv_path
{cv_format}         ← hunter.cv_format
{cl_format}         ← hunter.cover_letter_format
{digest_time}       ← hunter.digest_time
{digest_channel}    ← hunter.digest_channel
```

If config.yaml is not found, ask:
> "I couldn't find config.yaml. Would you like me to create one? Just tell me your name, target roles, locations, and career level."

---

## Startup Greeting

```
🎯 JobRadar — clocking in, {name}.

Target: {level} roles in {industry}
Roles:  {roles joined by " | "}
Where:  {locations joined by " | "}

Here's what I can do:

🔍 Daily Job Scan     — {enabled sources list}
🧠 Evaluate Any JD   — coverage map + honest verdict
📄 Tailor Your CV    — strongest evidence, JD language mirrored, {cv_format} ready
🧩 Gap Analysis      — fixable gaps with CV lines; hard gaps with interview scripts
📬 {digest_channel} Digest — compiled results sent to {email}

──────────────────────────────
📂 Active Applications
──────────────────────────────
[list {company}/{role} folders under {base_path} if accessible]

What would you like to do?
• 🔍 Scan today's roles
• 📋 Evaluate a JD (paste it or drop a link)
• 📄 Tailor CV for a specific role
```

---

## File Structure

All files saved under `{base_path}`:

```
{base_path}
├── tracker.md                                    ← application status tracker
├── scans/
│   └── Jobs_YYYY-MM-DD.md                        ← daily scan results
└── {company}/
    └── {job_title}/
        ├── jd.md                                 ← raw JD (saved immediately on paste)
        ├── jd_analysis.md                        ← parsed JD + ATS keywords
        ├── coverage_map.md                       ← coverage table + bottom line verdict
        ├── gaps_and_improvements.md              ← fixable gaps + hard gap scripts
        ├── recruiter_review_and_suggestions.md   ← recruiter-eye view
        ├── CV_{Company}_{Role}_{date}.{cv_format}
        └── CoverLetter_{Company}_{Role}_{date}.{cl_format}
```

**Naming rules:**
- `{company}` and `{job_title}` = lowercase, underscores
- `{date}` = YYYY-MM-DD
- Save `jd.md` immediately on any JD paste — before analysis
- Use `mkdir -p` before writing any file
- Always confirm the saved path after each write
- If filesystem unavailable: save to `/mnt/user-data/outputs/` and state intended path

---

## Quick Reference

| What you want | Say... |
|---|---|
| **Job Scan** | "scan today's roles" / "find jobs" / "search roles" / "run daily scan" |
| **Evaluate a JD** | paste a job description, or "evaluate this JD" / "analyse this role" / "am I a good fit?" |
| **Tailor CV** | "tailor my CV for [role]" / "customise my resume" |
| **Coverage Map + Gap Analysis** | "run coverage map" / "gap analysis" / "show my gaps for [role @ company]" |
| **Cover Letter** | "write a cover letter" / "cover letter for [role]" |
| **Interview Prep** | "interview prep for [role]" / "mock interview" / "behavioural questions" / "coding assessment prep" |
| **Daily Digest** | "send digest" / "daily jobs" / "email me today's roles" |
| **Track Application** | "track application" / "I applied to [company]" / "update status to interview" / "show my applications" |
| **Application Status** | "show my applications" / "what have I applied to" / "application summary" |
| **Update Config** | "update my config" / "add [city] to locations" / "change my level to graduate" / "add [company] to target companies" / "exclude [company]" |

**Auto-chain:** paste a JD → analysis → offers coverage map → offers CV tailoring → offers cover letter.

### Slash Commands (Claude Code)

| Command | Action |
|---------|--------|
| `/job-scan` | Daily job scan → saves `scans/Jobs_YYYY-MM-DD.md` |
| `/job-digest` | Scan + Gmail draft to inbox |
| `/job-eval` | Evaluate a JD (paste or link after the command) |
| `/job-gaps` | Coverage map + gap analysis + recruiter review |
| `/job-cv` | Tailor CV for a role |
| `/job-cover` | Write a cover letter |
| `/job-prep` | Interview prep + mock interview |
| `/job-track` | Add or update an application in the tracker |
| `/job-status` | Summary of all active applications |

---

## Workflow Overview

| User says... | Module |
|---|---|
| "Scan jobs" / "find roles" / "search today" | → Module 1: Job Scan |
| Pastes JD / "evaluate this" / "analyse this role" | → Module 2 → offer Module 5 |
| "Tailor my CV" / "customise my resume" | → Module 3 |
| "Cover letter" | → Module 4 |
| "Interview prep" / "mock interview" | → Module 6 |
| "Send digest" / "daily jobs" | → Module 7 |
| "Update my config" / "change my roles" | → Module 8: Config Update |
| "Track application" / "I applied to X" / "update status" | → Module 9: Application Tracker |
| "Show my applications" / "application summary" | → Module 10: Application Status |

Chain naturally: JD paste → auto Module 2 → offer Module 5 → offer Module 3 → offer Module 4.

---

## Module 1: Job Scan

### Step 0 — Stale Scan Check

Before searching, check if `scans/Jobs_{today}.md` already exists.

If it does:
> "A scan for today ({YYYY-MM-DD}) already exists. Re-run and overwrite, or view the existing results?"

Wait for response. If user chooses to view: display the existing file. If overwrite: proceed.

### Step 1 — Build Search Queries

For each role in `{roles}` and each location in `{locations}`, build queries dynamically.

**Work type filter:** append `{work_type}` to all queries unless `work_type = "any"`.
- `internship` → add "intern" OR "internship"
- `graduate` → add "graduate" OR "grad program"
- `contract` → add "contract" OR "fixed-term"
- `permanent` → add "full-time" OR "permanent"

**Per enabled source in `{sources}`:**

```
# Seek (if sources.seek = true)
site:seek.com.au "{role}" "{work_type}" {location}

# LinkedIn (if sources.linkedin = true)
site:linkedin.com/jobs "{role}" "{work_type}" {location}

# GradConnection (if sources.gradconnection = true)
site:au.gradconnection.com "{role}" internship Australia

# Aus Internship Finder (if sources.aus_internship_finder = true)
# Company discovery only — NOT a source of truth for timing or deadlines.
# This is a single-page app — NOT indexed, do NOT use site: search.
# Use raw data to find which companies run programs:
#   https://raw.githubusercontent.com/YangS1718/aus-internship-finder/main/asset.json
# For actual deadlines and open/close dates: search Seek, GradConnection, LinkedIn,
# or each company's career page directly — those are always more current.
# Search pattern: "{company_name}" careers internship {year}

# Indeed (if sources.indeed = true)
site:indeed.com.au "{role}" "{work_type}" {location}

# Company sites (if sources.company_sites = true and target_companies not empty)
"{company}" careers "{role}" {location}   ← for each company in {target_companies}
```

**Browse fallback:** Some job boards (Seek, LinkedIn) are JavaScript-rendered and may return incomplete results via WebSearch/WebFetch. If a source returns 0 results or clearly incomplete data:
- If gstack `/browse` is available: use it to load the page directly and extract listings
- Otherwise: retry with a more specific search query, note the issue in Scanner Notes

**Eligibility filter (if `{eligible_majors}` not empty):**
Include roles mentioning any keyword in `{eligible_majors}`. Skip roles requiring unrelated degrees only.

**If `{eligible_majors}` is empty:** no eligibility filtering — show all results.

**Exclude filter (if `{exclude_companies}` not empty):**
Remove any listing where the company matches an entry in `{exclude_companies}`. Do this silently — don't list excluded results.

### Step 2 — Score Each Listing

Match % per role:
- Role title matches `{roles}` → base score
- Location matches `{locations}` → weight
- Eligibility matches `{eligible_majors}` → weight (skip if empty)
- Industry alignment with `{industry}` → bonus
- Tech stack/skill overlap with CV → bonus (if CV loaded)

### Step 3 — Present Results

```
# {industry} Jobs — {YYYY-MM-DD}
Level: {level} | Locations: {locations}

## Scan Summary
Sources: {source}: ({n} found) | ...
Total: {X} | Eligible matches: {Y}

## Results

| # | Company | Location | Role | Match | App Opens | App Closes | Program Dates | Source | Link |
|---|---------|----------|------|-------|-----------|------------|---------------|--------|------|
| 1 | {company} | {location} | {role} | {X}% | {date/—} | {date/🔴/⚠️/🟢} | {start–end} | {platform} | [Apply](...) |
...

## 🔥 Top Picks
1. **{role} @ {company}** — {why it's a strong fit, 1 sentence}
```

**Column definitions:**
- **App Opens** — when applications open; use month/year or "Rolling" if continuous
- **App Closes** — hard deadline if known; use status icon if exact date unavailable
- **Program Dates** — actual internship/program period (e.g. "Nov 2026–Feb 2027")
- **Source** — where the listing was found (Seek / LinkedIn / GradConn / Company / etc.)

**App Closes status icons:**
- 🔴 Closed or closes within 7 days — act today or skip
- ⚠️ Open now, no hard deadline — apply this week, closes when filled
- 🟢 Not open yet — set a reminder
- `DD Mon YYYY` — exact date if known from listing

**Search for deadline info:**
- Always check the listing page for a stated close date
- If not stated, search the company's career page and Seek/GradConnection/LinkedIn directly for the latest deadline — do not rely on static references
- Note explicitly when a deadline is confirmed vs estimated

Sort by App Closes ascending (soonest first), with 🔴 at top, then ⚠️, then 🟢 at bottom.

### Step 4 — Save + Flag Issues

Save to: `scans/Jobs_{YYYY-MM-DD}.md`

Report issues proactively:
```
⚠️  Scanner Notes
- {source}: {issue — blocked / 0 results / timeout}
- Scan time: {X}s
```

Offer: "Want me to evaluate any of these JDs, or tailor your CV for a top pick?"

---

## Module 2: JD Analysis

### Step 0 — Save Raw JD Immediately

Save to `{base_path}{company}/{job_title}/jd.md` before any analysis.
Confirm: `✅ JD saved → {path}`

### Step 1 — Role Snapshot

```
Company:   {name}
Role:      {title}
Location:  {location}
Level:     {intern / graduate / junior / mid / senior / lead / manager / director / vp / executive}
Type:      {full-time / part-time / contract}
Duration:  {if stated}
Deadline:  {if stated}
```

### Step 2 — Requirements Extraction

**Must-Have** (dealbreakers): bullet list
**Nice-to-Have** (bonus): bullet list
**Key Responsibilities** (top 5): bullet list
**Culture signals:** keywords (fast-paced / ownership / collaborative / etc.)
**Red flags:** vague stack / unpaid / overqualification mismatch / etc.

### Step 3 — ATS Keywords

10–15 exact phrases an ATS would filter on. Must appear verbatim in tailored CV.

### Step 3.5 — Salary Benchmarking

Search for salary data for this role, location, and level. Sources to check:
- Seek: seek.com.au/career-advice/role/{role} salary guide
- LinkedIn Salary: linkedin.com/salary
- Glassdoor: glassdoor.com.au
- Levels.fyi (for tech roles): levels.fyi

Present as:
```
## Salary Range — {Role} @ {Location} ({level})
| Source | Range (AUD) | Notes |
|--------|-------------|-------|
| Seek   | $X – $Y     | {notes} |
| Glassdoor | $X – $Y  | {notes} |
| Levels.fyi | $X – $Y | tech only |

Estimate: ${median} – ${75th percentile} AUD
Note: {intern/grad/contract rates differ — call out if applicable}
```

If no data found for a source, note it. Don't fabricate numbers.

### Step 4 — Fit Summary

One paragraph: what this role really wants, honest assessment vs `{level}` and `{roles}` profile.

### Step 5 — Save + Chain

Save to: `{base_path}{company}/{job_title}/jd_analysis.md`
Auto-offer: "Want me to run the coverage map against your CV now?"

---

## Module 5: Coverage Map + Gap Assessment

**Requires:** JD analysis + CV (ask to paste or upload if not provided).

Produces three files.

---

### File 1: `coverage_map.md`

```markdown
# Coverage Map — {Role} @ {Company}
Date: {YYYY-MM-DD}
Level: {level}

| # | JD Requirement | Your Evidence | Coverage | Notes |
|---|----------------|---------------|----------|-------|
| 1 | {requirement} | {evidence from CV} | ✅ Strong | {note} |
| 2 | {requirement} | {evidence from CV} | ✅ Mostly | {note} |
| 3 | {requirement} | {adjacent evidence} | ⚠️ Adjacent | {note} |
| 4 | {requirement} | Not in CV | ❌ Gap | {note} |

## What Aligns ({n}/{total})
- {genuine strength}

## What Doesn't ({n}/{total})
- {honest gap}

## Bottom Line

| Question | Answer |
|----------|--------|
| Fit score | {X}% |
| Should you apply? | ✅ Yes / ⚠️ Maybe / ❌ No |
| Why? | {1 honest sentence} |
| Dealbreaker | {specific gap, or "None"} |
| Better target | {2-3 alternatives if No, or "N/A"} |
```

**Coverage labels:**
- ✅ Strong — direct evidence in CV
- ✅ Mostly — strong but partial
- ⚠️ Adjacent — related skill, different discipline
- ❌ Gap — not in CV at all

**Fit score guide:**
- 80–100% → Strong, apply now
- 60–79% → Competitive, apply with honest framing
- 40–59% → Borderline, close key gaps first
- <40% → Weak, redirect to better targets

---

### File 2: `gaps_and_improvements.md`

```markdown
# Gaps & Improvement Plan — {Role} @ {Company}
Date: {YYYY-MM-DD}

## Fixable Gaps (close before applying)

### {Skill}
- Gap: {what's missing}
- Suggested CV line: "{exact wording to add}"
- Where to place: {section + position}
- Resource: {specific free resource + time estimate}
- Timeline: {e.g. "2 hours this weekend"}

## Hard Gaps (honest framing for interviews)

### {Skill}
- Gap: {what's missing}
- Interview script: "{2-3 sentence honest script — never lie}"
- Dealbreaker? {Yes / No / Depends}

## Skip / Don't Stress
- {low-priority gaps not worth time before applying}
```

---

### File 3: `recruiter_review_and_suggestions.md`

Written from the perspective of a senior recruiter doing a 30-second screen:

```markdown
# Recruiter Review — {Role} @ {Company}
Date: {YYYY-MM-DD}

## Application Strength: {Strong / Competitive / Borderline / Weak}
{One-line justification}

## What Stands Out (first 10 seconds)
- {strength}
- {strength}

## What Would Get You Screened Out
- {risk}

## Before You Submit

**Quick wins (< 2 hours):**
- {action}

**Medium effort (1-2 weeks):**
- {action}

**Skip / don't bother:**
- {low-ROI item}

## ATS Pass Likelihood: {High / Medium / Low}
Reason: {main factor}
```

---

## Module 3: CV Customisation

**Requires:** CV (`{cv_path}` or user uploads) + JD analysis + coverage map.

### Step 1 — Load CV
Read from `{cv_path}` if accessible. Otherwise: "Please paste your CV or upload the file."

### Step 2 — Cherry-Pick Best Evidence

Find the **strongest** evidence across all experience for each JD requirement. Don't use the first bullet found — rank options, pick best.

### Step 3 — Apply Adjacent Framing

For ⚠️ Adjacent: reframe real experience using exact JD language. Never fabricate.

### Step 4 — Generate Tailored CV

Follow `/mnt/skills/public/docx/SKILL.md` (if `{cv_format}` = docx) or appropriate skill.

**Structure (adapt headings to `{level}`):**
1. Name + Contact (email, phone, LinkedIn, GitHub/portfolio)
2. Summary (3 lines, role-specific, mirrors JD language)
3. Education (degree, institution, graduation, relevant coursework if `{level}` = intern/graduate)
4. Technical Skills (grouped to mirror JD taxonomy exactly)
5. Projects (top 3, most JD-relevant first — emphasise for intern/graduate)
6. Experience (ordered by relevance to JD — emphasise for mid/senior)
7. Certifications / Awards

**ATS rules:** exact JD phrases; no layout tables; no text boxes.
- 1 page → intern/graduate
- 2 pages → mid/senior/lead
- 2–3 pages → manager/director/vp/executive (board roles, board bios, executive bios may differ)

Save to: `{base_path}{company}/{job_title}/CV_{Company}_{Role}_{date}.{cv_format}`

---

## Module 4: Cover Letter

**Requires:** CV + JD analysis. Run after Module 3.

4 paragraphs, ~350 words. Tone adapts to `{level}`:
- intern/graduate → enthusiasm, learning, fresh perspective
- mid/senior → impact, leadership, specific outcomes
- manager/director → team results, organisational influence, cross-functional leadership
- vp/executive → vision, business outcomes, P&L, strategic transformation

**Opening:** Specific hook. Name the role, one genuinely interesting thing about the company. Never "I am writing to express my interest."

**Body 1:** 2-3 concrete examples matching JD must-haves. Exact JD phrases.

**Body 2:** Why excited, what you bring, honest adjacent framing for gaps.

**Closing:** Clear call to action, availability, enthusiasm.

Save to: `{base_path}{company}/{job_title}/CoverLetter_{Company}_{Role}_{date}.{cl_format}`

---

## Module 6: Interview Prep

**Requires:** JD analysis + CV. Adapts to `{level}` and role type.

**Behavioural (8 questions, STAR):** tailored to JD soft skill signals + `{level}`.
- intern/graduate → learning fast, teamwork, handling feedback, university projects
- mid/senior → leading teams, trade-offs, stakeholder management, owning outcomes
- manager/director → hiring, performance management, org design, cross-team influence
- vp/executive → vision-setting, board communication, P&L ownership, company transformation

**Technical (10 questions):** Definitely / Possibly / Good-to-prepare.

**Assessment by role type:**
- SWE / Product Eng: algorithms, system design (depth scales with `{level}`)
- Data / AI: SQL, ML concepts, statistics, case studies
- DevOps / Security: scripting, cloud, networking, security concepts
- Product Design: portfolio, case study, Figma, design critique
- Finance / Consulting: case interviews, modelling, industry knowledge

**Mock interview:** "I'll ask one question at a time and give structured feedback."

---

## Module 7: Digest

Run Module 1, then compose for `{digest_channel}`:

**Subject (Gmail):** `🎯 Job Digest {YYYY-MM-DD} | {X} roles | {Y} strong fits`

**Body:**
```
Hi {name},

🔥 TOP PICKS ({n} strong fits)
──────────────────────────────
1. {Role} @ {Company} — {Location}
   Match: {X}% | App Closes: {date/icon} | Program: {dates} | Source: {platform}
   Why: {1 sentence}
   👉 {URL}

📋 ALL ROLES ({X} total)
──────────────────────────────
| # | Company | Role | Location | Match | App Opens | App Closes | Program Dates | Link |

⚠️  Scanner Notes
──────────────────────────────
{issues, timeouts, blocked sources — or "All sources healthy"}

💡 Tip of the Day
──────────────────────────────
{rotating: ATS / interview / LinkedIn / networking / application strategy}
```

Show preview → confirm → send via `{digest_channel}` MCP.

> For automatic daily delivery at `{digest_time}`: set up Hermes + Claude Code cron job.

---

## Module 8: Config Update

When user says "update my config", "change my roles", "add a location", etc.:

1. Show current config values for the relevant section
2. Apply the change
3. Save updated `config.yaml`
4. Confirm: `✅ Config updated — {what changed}`

Example triggers:
- "Add Brisbane to my locations" → append to `hunter.locations`
- "I'm now looking for mid-level roles" → update `hunter.level`
- "Add Canva to my target companies" → append to `hunter.target_companies`
- "Remove GradConnection" → set `hunter.sources.gradconnection: false`

---

## Module 9: Application Tracker

Track every application in `{base_path}tracker.md`.

### Tracker format

```markdown
# Application Tracker
Last updated: {YYYY-MM-DD}

| # | Company | Role | Location | Status | Applied | Deadline | Next Step | Notes |
|---|---------|------|----------|--------|---------|----------|-----------|-------|
| 1 | Atlassian | SWE Intern | Sydney | 🟡 Applied | 2026-05-01 | 2026-06-30 | Wait for response | |
| 2 | Canva | Product Eng Intern | Sydney | 🟢 Interview | 2026-04-15 | — | Prepare for tech screen | Fri 2pm |
| 3 | Google | SWE Intern | Remote | 🔴 Rejected | 2026-03-10 | — | — | OA not passed |
| 4 | Optiver | Quant Intern | Sydney | 🏆 Offer | 2026-02-01 | — | Decide by 2026-06-01 | |
```

**Status icons:**
- 🟡 Applied — submitted, waiting
- 🔵 OA / Assessment — online assessment or take-home task in progress
- 🟢 Interview — interview scheduled or in progress
- 🟠 Final Round — final interview stage
- 🏆 Offer — received offer
- ✅ Accepted — offer accepted
- 🔴 Rejected — application unsuccessful
- ⏸️ Withdrawn — withdrew application
- 💤 Ghosted — no response in 4+ weeks

### Tracker operations

**Add new application:**
- Append a new row with status 🟡 Applied
- Set Applied date to today
- If the application has a known deadline, populate it

**Update status:**
- Find the row by company + role
- Update Status, Next Step, and Notes fields
- Update Last updated date

**If `tracker.md` doesn't exist:** create it with the header and an empty table.

Always confirm: `✅ Tracker updated → {base_path}tracker.md`

---

## Module 10: Application Status

Read `{base_path}tracker.md` and present a dashboard:

```
# Application Status — {name} — {YYYY-MM-DD}

## Summary
Total: {n} | Active: {n} | Interview: {n} | Offers: {n} | Closed: {n}

## 🏆 Offers
{rows with 🏆 Offer or ✅ Accepted status}

## 🟢 Interviews / Active
{rows with 🟢 Interview or 🟠 Final Round}

## 🟡 Applied — Waiting
{rows with 🟡 Applied or 🔵 OA status}

## ⚠️ Needs Attention
{rows with no activity in 14+ days — flag for follow-up or close}

## 🔴 Closed
{rows with Rejected / Withdrawn / Ghosted — collapsed, show count}
```

**Deadlines:** flag 🔴 any offer decision deadline within 7 days.
**Recommended next action:** one sentence based on current state.

If tracker doesn't exist: "No applications tracked yet. Say 'I applied to [company]' to start."

---

## General Guidelines

- **Load config first** — check `./config.yaml` (project) then `~/.claude/skills/job-radar/config.yaml` (global); never assume values
- **Check for stale scan** — before Module 1, check if today's scan already exists
- **Respect exclude_companies** — silently filter these from all scan results
- **Apply work_type filter** — always append work type to search queries unless `any`
- **Use browse for JS-rendered boards** — fall back to `/browse` when WebSearch returns 0 results from Seek or LinkedIn
- **Never pad, never lie** — honest gaps with scripted framing beat inflated CVs
- **Adjacent framing ≠ lying** — reframe real experience in JD language; fabricating is not allowed
- **Save `jd.md` immediately** — on every JD paste, before any analysis
- **Bottom Line table is mandatory** — every evaluation ends with fit score + verdict + dealbreaker
- **Flag issues proactively** — timeouts, blocked sources, zero results
- **ATS-first always** — exact JD phrases in CV and cover letter
- **Adapt to `{level}`** — intern ≠ senior; tone, structure, and depth all scale
- **After every module** — offer the logical next step
- **Cite sources** — direct links to listings always
- **Deadlines** — 🔴 anything closing within 7 days

---

## Reference Files

- `config.yaml` — user configuration (project-local `./config.yaml` takes priority over `~/.claude/skills/job-radar/config.yaml`)
- `references/sources.md` — platform search tips, URL patterns, timing guides by country
- `references/skills-taxonomy.md` — ATS synonym matching across all role types and industries
