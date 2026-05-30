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
{university}        ← hunter.university (string, optional)
{majors}            ← hunter.majors (list, optional)
{graduation_year}   ← hunter.graduation_year (integer, optional)
{level}             ← hunter.level
{roles}             ← hunter.roles (list)
{eligible_majors}   ← hunter.eligible_majors (list, may be empty)
{locations}         ← hunter.locations (list)
{industry}          ← hunter.industry (list — one or more values)
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

### Step 0.1 — Validate Config

After loading, validate before proceeding. Check every rule below. If **any** rule fails, stop and output the full error block — do not attempt to run the requested module.

**Required fields (must be present and non-empty):**
- `hunter.name` — non-empty string
- `hunter.email` — non-empty string, contains `@`
- `hunter.level` — must be one of: `intern | graduate | junior | mid | senior | lead | manager | director | vp | executive`
- `hunter.roles` — non-empty list (at least 1 entry)
- `hunter.locations` — non-empty list (at least 1 entry)
- `hunter.industry` — non-empty list; each entry must be one of: `tech | finance | accounting | consulting | healthcare | government | education | energy | retail | legal | marketing | hr | construction | manufacturing | any`
- `hunter.work_type` — must be one of: `internship | graduate | contract | permanent | any`
- `hunter.base_path` — non-empty string, should end with `/`

**Optional but typed (validate if present):**
- `hunter.university` — if present, must be a string
- `hunter.majors` — if present, must be a list of strings
- `hunter.graduation_year` — if present and not null, must be a 4-digit integer between 1980 and 2100
- `hunter.sources` — if present, must be a map; each value must be `true` or `false`
- `hunter.digest_time` — if present, must match `HH:MM` format (24h)
- `hunter.digest_channel` — if present, must be one of: `gmail | slack | none`
- `hunter.cv_format` — if present, must be one of: `docx | pdf`
- `hunter.cover_letter_format` — if present, must be one of: `docx | pdf`

**Error output format:**
```
❌ config.yaml has errors — fix these before running JobRadar:

  • hunter.level: "senoir" is not a valid level. Must be one of: intern | graduate | junior | mid | senior | lead | manager | director | vp | executive
  • hunter.industry: "fintech" is not a valid industry. Must be one of: tech | finance | ...
  • hunter.roles: is empty — add at least one target role

Run /job-setup to fix your config interactively, or edit config.yaml directly.
```

If validation passes, proceed silently — no success message.

---

## Startup Greeting

**Step 1 — Load status data (silent):**
- Read `{base_path}tracker.json` if it exists
- Compute: `{active}` = entries with status in (watchlist, applied, oa, interview, final_round, offer)
- Compute: `{closing_soon}` = entries where `deadline` is within 7 days and status not in (rejected, withdrawn, ghosted, accepted)
- Compute: `{interviews}` = entries with status in (interview, final_round)
- Read `seen.json` if it exists → get `{seen_count}` = length of hashes array
- Check `scans/` folder → find most recent scan file → extract date as `{last_scan}`

**Step 2 — Render greeting:**

```
🎯 JobRadar — {name}

{if active > 0}
  📊 {active} active application{s}  {if interviews > 0}· {interviews} at interview stage{end}
  {if closing_soon entries exist}
  ⚠️  Closing soon: {closing_soon entries as "Company — Role (deadline)"}
  {end}
  Last scan: {last_scan date or "never"}
{else}
  No applications tracked yet — run /job-scan to find your first roles.
{end}

Here's what I can do:
🔍 /job-scan       — search all job boards + company career pages
🧠 /job-eval       — honest fit score, ATS keywords, Bottom Line verdict
📄 /job-cv         — tailor CV to a specific JD
🧩 /job-gaps       — fixable gaps + hard gap interview scripts
✉️  /job-cover      — cover letter, 4 paragraphs, role-specific
🎤 /job-prep       — STAR behavioural + technical prep + mock interview
📝 /job-oa         — OA prep: coding, video interview, psychometric, case study
📬 /job-digest     — send Gmail digest with today's top picks
📊 /job-track      — add or update an application
🗂️  /job-status     — full dashboard of all applications
🤝 /job-network    — alumni map + personalised outreach drafts
✉️  /job-followup   — draft follow-up or thank-you email
⚙️  /job-setup      — update config (roles, locations, level, companies)

What's first?
```

**Conciseness rule:** if `{active}` = 0 and `{last_scan}` = "never", skip the stats block entirely and go straight to the command list. Don't pad with zeros.

---

## File Structure

All files saved under `{base_path}`:

```
{base_path}
├── tracker.json                                  ← application tracker (source of truth)
├── tracker.md                                    ← auto-generated view (do not edit)
├── networking/
│   └── alumni_map.json                           ← alumni networking data (gitignored)
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
| `/job-oa` | OA prep — coding, video, psychometric, case study |
| `/job-track` | Add or update an application in the tracker |
| `/job-status` | Summary of all active applications |
| `/job-network` | Alumni map + personalised outreach drafts |

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
| "Find alumni" / "networking" / "who from my school works at X" | → Module 11: Alumni Networking |

Chain naturally: JD paste → auto Module 2 → offer Module 5 → offer Module 3 → offer Module 4.

---

## Module 1: Job Scan

### Step 0 — Stale Scan Check

Before searching, check if `scans/Jobs_{today}.md` already exists.

If it does:
> "A scan for today ({YYYY-MM-DD}) already exists. Re-run and overwrite, append as a new run, or view existing results?"

- **View**: display the existing file, stop.
- **Overwrite**: proceed, save to `scans/Jobs_{YYYY-MM-DD}.md` (replaces existing).
- **New run**: save to `scans/Jobs_{YYYY-MM-DD}_{HH-MM}.md` using current time (e.g. `Jobs_2026-05-26_14-32.md`). Preserves the original.

### Step 0.5 — Load Dedup Index

Load `seen.json` from the project root.

If file doesn't exist, initialise in memory (do not write yet):
```json
{ "last_updated": "{today}", "hashes": [] }
```

**Fingerprint format** — one string per listing, built at search time:
- Primary (when URL is available): `{source_key}|{url}` — e.g. `seek|https://www.seek.com.au/job/78291047`
- Fallback (no URL): `{source_key}|{company-slug}|{role-slug}` — e.g. `linkedin|atlassian|swe-intern-2026`

Source keys: `seek`, `linkedin`, `gradconnection`, `indeed`, `company`, `aus_internship_finder`

Slugs: lowercase, hyphens only, drop punctuation. `"SWE Intern 2026/27"` → `swe-intern-2026`.

Store the loaded hash set in memory for Step 2 filtering.

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

### Step 2 — Score and Deduplicate

For each listing found across all sources:

**2a — Build fingerprint** using the format defined in Step 0.5.

**2b — Dedup check:**
- If the fingerprint exists in the loaded hash set → mark listing as `[seen]`, exclude from Results table.
- If not in hash set → mark as `[new]`, include in Results table.
- Track all `[new]` fingerprints in a separate list for Step 4.

**2c — Score each `[new]` listing:**
- Role title matches `{roles}` → base score
- Location matches `{locations}` → weight
- Eligibility matches `{eligible_majors}` → weight (skip if empty)
- Industry alignment with `{industry}` → bonus
- Tech stack/skill overlap with CV → bonus (if CV loaded)

**2d — Report dedup stats** in the Scan Summary table:
`{X} new | {Y} already seen (hidden)` — so user knows what was filtered.

### Step 3 — Present Results

Use this exact structure every run — do not add extra sections, urgency boxes, or duplicate tables:

```
# {industry joined by " / "} Jobs — {YYYY-MM-DD}
Level: {level} | Work type: {work_type} | Locations: {locations joined by " | "}
Roles: {roles joined by " | "}

## Scan Summary
| Source | Status | Notes |
|--------|--------|-------|
| {source} | ✅ OK / ⚠️ Partial / ❌ Failed | {notes} |

Total found: {X} | New: {N} | Already seen: {S} (hidden) | Actionable (open now): {Y} | Not yet open: {Z} | Closed: {C}

---

## Results

| # | Company | Location | Role | Match | App Opens | App Closes | Program Dates | Source | Link |
|---|---------|----------|------|-------|-----------|------------|---------------|--------|------|
| 1 | {company} | {location} | {role} | {X}% | {date/—} | {date/🔴/⚠️/🟢} | {start–end} | {platform} | [Apply/View](...) |
...

---

## 🔥 Top Picks

1. **{role} @ {company}** — {why it's a strong fit, 1 sentence}. {call to action: "Apply today." / "Set a reminder for {date}."}

---

## ⚠️ Scanner Notes

- {source}: {issue — blocked / 0 results / timeout / estimated deadline}
- Scan time: ~{X} min
```

**Key rules:**
- Closed listings stay in the main Results table (with 🔴 Closed {date}) — do NOT move them to a separate section
- No extra "This Week" / "Act Now" / "Urgent" boxes — urgency is conveyed by 🔴 in the table and ordering
- Top Picks: one sentence why + one call to action. No paragraph descriptions.

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

### Step 4 — Save Scan + Update Dedup Index

**4a — Save scan file** to: `scans/Jobs_{YYYY-MM-DD}.md` (or timestamped variant per Step 0).

**4b — Update seen.json:**
- Take the list of `[new]` fingerprints collected in Step 2.
- Append them to the `hashes` array in the loaded seen.json object.
- Set `last_updated` to today.
- Write the full updated object back to `seen.json`.
- Do not deduplicate the hashes array itself — duplicates are harmless and writes are faster.

**4c — Confirm:**
```
✅ Scan saved → scans/Jobs_{YYYY-MM-DD}.md
✅ seen.json updated — {N} new fingerprints added ({total} total)
```

Report source issues proactively:
```
⚠️  Scanner Notes
- {source}: {issue — blocked / 0 results / timeout}
- Scan time: ~{X} min
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

Run Module 1, then send via `{digest_channel}` MCP. Do not ask for confirmation.

**Subject:** `🎯 Job Digest {YYYY-MM-DD} | {X} roles | {Y} strong fits`

**Build the email as HTML.** Use the CSS constants below — copy them exactly, never invent new styles.

### CSS constants (frozen — do not modify)

```
Outer div:       font-family:Arial,sans-serif;font-size:14px;color:#222;max-width:760px;margin:auto;padding:16px
H2 (title):      color:#1a1a2e;margin-bottom:4px
Callout box:     background:#fff3cd;padding:10px 14px;border-left:4px solid #ffc107;margin:16px 0
Divider:         border:none;border-top:1px solid #ddd;margin:20px 0
TOP PICKS h3:    color:#c0392b
TOP PICKS table: width:100%;border-collapse:collapse;font-size:13px
TOP PICKS rows:  odd → border-bottom:1px solid #eee  |  even → border-bottom:1px solid #eee;background:#fafafa
TOP PICKS cells: padding:8px
ALL ROLES table: width:100%;border-collapse:collapse;font-size:11px
ALL ROLES rows:  closed → background:#ffe0e0  |  open/rolling → background:#fff9e0  |  not yet open → background:#e8f5e9
ALL ROLES cells: padding:5px
Header row:      background:#f0f0f0
Footer:          font-size:11px;color:#888
Tip paragraph:   font-size:13px
Scanner notes:   font-size:13px
```

### Email structure

**Outer wrapper:**
```html
<div dir="ltr"><u></u>
<div style="font-family:Arial,sans-serif;font-size:14px;color:#222;max-width:760px;margin:auto;padding:16px">
  ...content...
</div></div>
```

**Header + intro:**
```html
<h2 style="color:#1a1a2e;margin-bottom:4px">🎯 Job Digest — {YYYY-MM-DD}</h2>
<p style="margin-top:4px">Hi {name},</p>
<p>Daily scan across {sources}. <strong>{X} roles tracked</strong> — {N} open now, {N} not yet open, {N} closed[, {N} new finds].</p>
<!-- {sources} = comma-separated list of ALL enabled sources from hunter.sources, e.g. "Seek, LinkedIn, GradConnection, Aus Internship Finder, Indeed, and direct career pages" — never omit any enabled source -->
```

**Act This Week** — include only if rolling or imminent-close roles exist:
```html
<p style="background:#fff3cd;padding:10px 14px;border-left:4px solid #ffc107;margin:16px 0">
  <strong>⚡ Act this week:</strong> {1–2 sentences. Name roles, why urgent, any hard deadlines.}
</p>
```

**Divider between every section:**
```html
<hr style="border:none;border-top:1px solid #ddd;margin:20px 0">
```

**TOP PICKS — open/EoI roles only (4–8 rows), no closed:**
```html
<h3 style="color:#c0392b">🔥 TOP PICKS — {N} strong fits</h3>
<table style="width:100%;border-collapse:collapse;font-size:13px">
  <tr style="background:#f0f0f0">
    <th style="padding:8px;text-align:left">#</th>
    <th style="padding:8px;text-align:left">Role @ Company</th>
    <th style="padding:8px;text-align:left">Location</th>
    <th style="padding:8px;text-align:left">Match</th>
    <th style="padding:8px;text-align:left">Closes</th>
    <th style="padding:8px;text-align:left">Program</th>
    <th style="padding:8px;text-align:left">Why</th>
    <th style="padding:8px;text-align:left">Link</th>
  </tr>
  <!-- odd rows: style="border-bottom:1px solid #eee" -->
  <!-- even rows: style="border-bottom:1px solid #eee;background:#fafafa" -->
  <!-- role cell: <td style="padding:8px"><strong>Role @ Company</strong></td> -->
  <!-- link cell: <td style="padding:8px"><a href="URL" target="_blank">Apply</a></td> -->
</table>
```
- Why cell: 1 sentence, specific — tech stack, firm prestige, program structure. No filler.
- Link text: "Apply" if open · "EoI" if expression of interest

**ALL ROLES — every listing including closed:**
```html
<h3>📋 ALL ROLES — {X} total</h3>
<table style="width:100%;border-collapse:collapse;font-size:11px">
  <tr style="background:#f0f0f0">
    <th style="padding:5px">Company</th><th style="padding:5px">Role</th>
    <th style="padding:5px">Location</th><th style="padding:5px">Match</th>
    <th style="padding:5px">Closes</th><th style="padding:5px">Program</th>
    <th style="padding:5px">Link</th>
  </tr>
  <!-- closed:        <tr style="background:#ffe0e0"> -->
  <!-- open/rolling:  <tr style="background:#fff9e0"> -->
  <!-- not yet open:  <tr style="background:#e8f5e9"> -->
  <!-- cells: <td style="padding:5px">VALUE</td> -->
</table>
```
- Link text: "Apply" · "Watch" · "EoI" · "View" by status
- Closes icons: 🔴 Closed {date} · ⚠️ Rolling · 🟢 ~{Mon YYYY} · `DD Mon YYYY` hard date (bold if ≤30 days)

**Scanner Notes:**
```html
<h3>⚠️ Scanner Notes</h3>
<ul style="font-size:13px">
  <li><strong>Company</strong>: note — per-company context, new finds (✨ new), blocked sources.</li>
</ul>
```

**Tip of the Day:**
```html
<h3>💡 Tip of the Day</h3>
<p style="font-size:13px"><strong>{Tip title}:</strong> {1 actionable paragraph tailored to roles in this scan. Rotate: quant prep / ATS / interview strategy / networking / sequencing.}</p>
```

**Footer:**
```html
<p style="font-size:11px;color:#888">Generated by JobRadar · {YYYY-MM-DD} · <a>View full scan</a></p>
```

---

> For automatic daily delivery at `{digest_time}`: set up a Claude Code cron job.

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

Source of truth: `{base_path}tracker.json`. `tracker.md` is a read-only view — always generated from JSON, never edited directly.

### tracker.json structure

```json
{
  "last_updated": "YYYY-MM-DD",
  "applications": [
    {
      "id": "atlassian-swe-intern-2026",
      "company": "Atlassian",
      "role": "SWE Intern 2026/27",
      "location": "Sydney",
      "status": "applied",
      "applied_date": "2026-05-01",
      "deadline": "2026-06-30",
      "source": "GradConnection",
      "url": "https://au.gradconnection.com/...",
      "match_score": 90,
      "next_step": "Wait for response",
      "outcome": null,
      "outcome_date": null,
      "interview_date": null,
      "offer_deadline": null,
      "rejection_reason": null,
      "start_date": null,
      "notes": ""
    }
  ],
  "networking": [
    {
      "id": "sarah-chen-atlassian-2026",
      "name": "Sarah Chen",
      "company": "Atlassian",
      "role": "Software Engineer",
      "university": "University of Melbourne",
      "major": "Computer Science",
      "graduation_year": 2024,
      "linkedin_url": "https://linkedin.com/in/sarah-chen",
      "status": "drafted",
      "contacted_date": null,
      "reply_status": "not_sent",
      "related_application_id": "atlassian-swe-intern-2026",
      "last_message_summary": "Asked for advice about SWE internship path",
      "notes": ""
    }
  ]
}
```

**Status enum** (machine values — map from user language):
- `watchlist` — saved, not yet applied
- `applied` — submitted, waiting
- `oa` — online assessment / take-home in progress
- `interview` — interview scheduled or in progress
- `final_round` — final interview stage
- `offer` — received offer
- `accepted` — offer accepted
- `rejected` — application unsuccessful
- `withdrawn` — withdrew application
- `ghosted` — no response in 4+ weeks

**Networking status enum** (`networking[].status`):
- `drafted` — message drafted, not sent
- `sent` — user sent message manually
- `replied` — alumni replied
- `meeting_scheduled` — coffee chat / call scheduled
- `referred` — referral or concrete intro offered
- `no_reply` — no reply after follow-up window

**ID generation:** lowercase hyphenated slug — `{company}-{role-keywords}-{year}`. Keep short and unique. Example: `atlassian-swe-intern-2026`, `canva-product-eng-intern-2026`.

### Tracker operations

**Step 1 — Load tracker.json:**
- Read `{base_path}tracker.json`
- If file doesn't exist, initialise: `{"last_updated": "{today}", "applications": [], "networking": []}`
- If an older tracker exists without `networking`, treat `networking` as an empty array and add it on next write.

**Step 2 — Add or update:**

*Add new application:*
- Generate an `id` slug from company + role + year
- Create a new entry with `status: "applied"`, `applied_date: today`, and all known fields
- For any unknown field (source, url, match_score, deadline) set to `null`
- Append to `applications` array

*Update existing application:*
- Find entry by matching `company` (case-insensitive) + `role` (partial match OK)
- Update only the fields the user mentioned: `status`, `next_step`, `notes`, `outcome`, `deadline`
- Leave all other fields unchanged

**Status transition rules — auto-populate on status change:**

| New status | Auto-set fields | Ask user (if not provided) |
|------------|----------------|---------------------------|
| `applied` | `applied_date: today` | — |
| `oa` | `next_step: "Complete assessment"` | deadline if not set |
| `interview` | `next_step: "Prepare for interview"` | `interview_date` |
| `final_round` | `next_step: "Final round prep"` | `interview_date` |
| `offer` | `outcome: "offer received"`, `outcome_date: today` | `offer_deadline` |
| `accepted` | `outcome: "accepted"`, `outcome_date: today` | `start_date` |
| `rejected` | `outcome: "rejected"`, `outcome_date: today` | `rejection_reason` (optional — "no reason given" if skipped) |
| `withdrawn` | `outcome: "withdrawn"`, `outcome_date: today` | — |
| `ghosted` | `outcome: "no response"`, `outcome_date: today` | — |

After a terminal status (`accepted`, `rejected`, `withdrawn`, `ghosted`), suggest: `💡 Want to run /job-followup to send a thank-you or follow-up email?`

**Step 3 — Write tracker.json:**
- Set `last_updated` to today
- Write the full updated JSON back to `{base_path}tracker.json`

**Step 4 — Regenerate tracker.md:**
- Generate `{base_path}tracker.md` from the current tracker.json data
- Format:

```markdown
# Application Tracker
Last updated: {YYYY-MM-DD}
> Auto-generated from tracker.json — do not edit directly.

| # | Company | Role | Location | Status | Applied | Deadline | Next Step | Notes |
|---|---------|------|----------|--------|---------|----------|-----------|-------|
| 1 | Atlassian | SWE Intern 2026/27 | Sydney | 🟡 Applied | 2026-05-01 | 2026-06-30 | Wait for response | |
```

Status icon mapping for tracker.md display:
- `watchlist` → 👁️ Watchlist
- `applied` → 🟡 Applied
- `oa` → 🔵 Assessment
- `interview` → 🟢 Interview
- `final_round` → 🟠 Final Round
- `offer` → 🏆 Offer
- `accepted` → ✅ Accepted
- `rejected` → 🔴 Rejected
- `withdrawn` → ⏸️ Withdrawn
- `ghosted` → 💤 Ghosted

Sort order in tracker.md: active first (watchlist → applied → oa → interview → final_round → offer), then closed (accepted → rejected → withdrawn → ghosted).

Always confirm: `✅ Tracker updated → {base_path}tracker.json (tracker.md regenerated)`

### Networking record operations

Module 11 owns `networking[]`, but Module 9 defines the storage contract.

When adding or updating a networking record:
- Find by `linkedin_url` first; fallback to `name + company` case-insensitive.
- Preserve original profile fields (`name`, `company`, `role`, `university`, `graduation_year`) unless the user explicitly corrects them.
- Update only interaction fields: `status`, `contacted_date`, `reply_status`, `related_application_id`, `last_message_summary`, `notes`.
- Set `last_updated` to today.
- Do **not** render networking rows into `tracker.md`; keep tracker.md application-focused.

---

## Module 10: Application Status

Read `{base_path}tracker.json` (not tracker.md) and present a live dashboard.

**Step 1 — Load:** Read `{base_path}tracker.json`. If missing: "No applications tracked yet. Say 'I applied to [company]' to start." Stop.

**Step 2 — Compute stats:**
- Total, Active (non-closed statuses), Interview (interview + final_round), Offers (offer + accepted), Closed (rejected + withdrawn + ghosted)
- Stale: entries where applied_date is 14+ days ago and status is still `applied` or `oa`
- Networking: count `networking[]` by status (`drafted`, `sent`, `replied`, `meeting_scheduled`, `referred`, `no_reply`)

**Step 3 — Present dashboard:**

```
# Application Status — {name} — {YYYY-MM-DD}

## Summary
Total: {n} | Active: {n} | Interview: {n} | Offers: {n} | Closed: {n}
Networking: Drafted {n} | Sent {n} | Replied {n} | Meetings {n} | Referrals {n}

## 🏆 Offers
{entries with status offer or accepted — show company, role, deadline, next_step}

## 🟢 Interviews / Final Round
{entries with status interview or final_round — sorted by deadline asc}

## 🟡 Applied — Waiting
{entries with status applied or oa — sorted by applied_date asc}

## 👁️ Watchlist
{entries with status watchlist}

## ⚠️ Needs Attention
{stale entries — no status change in 14+ days; flag for follow-up or close}

## 🔴 Closed
Rejected: {n} | Withdrawn: {n} | Ghosted: {n}
(type /job-status closed to see full list)

## 🤝 Networking
{if networking exists: show top 5 active contacts with company, status, next step; else "No alumni contacts tracked yet."}
```

**Deadlines:** flag 🔴 any `deadline` or offer decision within 7 days.
**Recommended next action:** one sentence based on current state (e.g. "You have 2 interviews this week — focus on prep.")

If an entry has `match_score`, show it as a small indicator: `[90%]`.

---

## Module 11: Alumni Networking

Help the user discover where similar alumni work, choose who to contact, and draft authentic outreach. This module uses `hunter.university`, `hunter.majors`, and `hunter.graduation_year` from config.

**Privacy rule:** save alumni data only under `networking/`. Never write alumni names, LinkedIn URLs, or outreach notes to tracked docs.

**Safety rule:** never send LinkedIn messages automatically. Draft only; the user reviews and sends manually.

### Command modes

| Mode | Purpose |
|------|---------|
| `/job-network --spike` | Validate whether LinkedIn search data is extractable in the current browser/session |
| `/job-network --map` | Build `networking/alumni_map.json` grouped by current company |
| `/job-network --list [company]` | Read saved map and show contact candidates |
| `/job-network --reach {company}` | Draft personalised outreach for selected alumni |
| `/job-network --prep {name}` | Prepare coffee chat questions and referral ask script |

### Preflight

Before any mode except `--list`, validate:
- `hunter.university` exists and is non-empty
- `hunter.majors` exists and has at least one entry
- `hunter.graduation_year` is a valid 4-digit year

If missing, stop with:

```
I need your alumni search profile first:
- university
- majors
- graduation_year

Run /job-setup or say "set my university to ..." and I’ll update config.yaml.
```

### `--spike` — LinkedIn extractability check

Goal: decide whether Chrome/LinkedIn extraction is viable before building automation.

Steps:
1. Ask user to confirm they are logged into LinkedIn in Chrome.
2. Build one manual search query using the first major:
   `site:linkedin.com/in "{university}" "{major}" "{graduation_year}" "Software Engineer" Australia`
3. If browser tooling is available, navigate to LinkedIn people search or Google results and inspect whether names, titles, companies, and URLs are visible as text.
4. Record result in `networking/spike_report.md`.

Report format:

```markdown
# LinkedIn Networking Spike — {YYYY-MM-DD}

## Result
Pass / Partial / Fail

## Tested query
{query}

## Extractability
| Field | Result | Notes |
|-------|--------|-------|
| Name | pass/partial/fail | |
| Current company | pass/partial/fail | |
| Role/title | pass/partial/fail | |
| LinkedIn URL | pass/partial/fail | |

## Decision
- If Pass: proceed with /job-network --map
- If Partial: use Google `site:linkedin.com/in` fallback and limit output confidence
- If Fail: do not automate; provide manual search queries only
```

### `--map` — Alumni company map

Create `networking/alumni_map.json`.

Search scope:
- University: `{university}`
- Majors: each value in `{majors}`
- Graduation year range: `{graduation_year - 2}` to `{graduation_year}`
- Location bias: Australia unless user asks otherwise

For each result, extract:
- `name`
- `current_company`
- `current_role`
- `graduation_year` if visible
- `linkedin_url`
- `source_query`

Deduplicate by `linkedin_url`; fallback to lowercase `name + current_company` if URL unavailable.

Output JSON:

```json
{
  "generated_date": "YYYY-MM-DD",
  "university": "University of Melbourne",
  "majors": ["Computer Science", "Software Engineering"],
  "graduation_years": [2023, 2024, 2025],
  "total": 0,
  "by_company": {}
}
```

Then present the top companies by alumni count and ask which company the user wants to inspect.

### `--list [company]`

Read `networking/alumni_map.json`. If missing, ask user to run `/job-network --map` first.

Show:
- company
- alumni count
- name
- current role
- graduation year if known
- contact status: untouched, drafted, sent, replied, meeting_scheduled, referred, no_reply

### `--reach {company}`

Read `networking/alumni_map.json`, filter to the company, and show 3-5 best candidates.

For selected alumni:
1. Inspect profile context if available.
2. Draft a message with a real icebreaker.
3. Keep connection request under 300 characters; InMail/email under 500 words.
4. End with a light ask: advice, coffee chat, or a few questions. Do not directly demand a referral in the first message.

After drafting, ask whether to mark the contact as `drafted` in `alumni_map.json`.

If user says yes:
- Update the matching person in `networking/alumni_map.json`:
  - `contacted`: false
  - `reply_status`: `drafted`
  - `last_message_summary`: one sentence
- Add or update the matching record in `{base_path}tracker.json` under `networking[]`:
  - `status`: `drafted`
  - `reply_status`: `not_sent`
  - `contacted_date`: null
  - `related_application_id`: best matching application at same company if one exists, otherwise null
- Confirm: `✅ Outreach draft tracked → tracker.json networking[]`

When user later says they sent, got a reply, scheduled a chat, or received a referral, update both:
- `networking/alumni_map.json` contact fields
- `{base_path}tracker.json` `networking[]` record

### `--prep {name}`

Generate:
- 5 coffee chat questions
- 1 intro sentence
- 1 graceful referral ask for the end of the conversation
- 1 thank-you follow-up message

---

## Module 12: Follow-up & Thank-you Emails

Triggered by `/job-followup` or when user says "send a follow-up", "write a thank-you", "follow up on my application", "14 days no response".

### Two modes

**Mode A — Follow-up after no response (applied / oa stage)**

Trigger conditions:
- User asks to follow up on a specific application, OR
- `/job-status` detects an entry stale for 14+ days with status `applied` or `oa`

Steps:
1. Load tracker.json, find the application
2. Check days since `applied_date` — if < 7 days, advise waiting
3. Generate a short, professional follow-up email:
   - Subject: `Following up — {role} application`
   - Body: 3 sentences max — reference application, express continued interest, polite ask for update
   - Tone: confident, not apologetic
4. Present draft, ask user to confirm before sending
5. If user confirms sent: update `next_step` in tracker.json to "Awaiting response after follow-up"

**Mode B — Thank-you after interview**

Trigger conditions:
- User says "thank-you email", "send thanks after interview", or status just changed to `interview` / `final_round`

Steps:
1. Load tracker.json, find the application
2. Ask: "Who did you interview with? Any specific topics to reference?"
3. Generate thank-you email:
   - Subject: `Thank you — {role} interview`
   - Body: 4 sentences — thank interviewer by name, reference one specific topic from the interview, restate enthusiasm, close lightly
   - Tone: warm, specific, not generic
4. Present draft for user review — never send automatically
5. If user confirms sent: update `next_step` to "Thank-you sent, awaiting next steps"

### Stale alert integration

When `/job-status` or startup greeting detects stale applications (14+ days, status `applied` or `oa`):

```
⚠️  {company} — {role}: applied {n} days ago, no update
    → Run /job-followup to draft a follow-up email
```

### Output format

```
✉️  Follow-up draft — {company} · {role}

Subject: {subject}

{body}

---
Send this? Once you've sent it, tell me and I'll update your tracker.
```

---

## Module 13: OA Preparation

Triggered by `/job-oa` or when user says "OA prep", "online assessment", "coding test", "HireVue", "psychometric", "aptitude test".

### Step 1 — Detect OA Type from JD

Read the JD (from `jd.md` or pasted content). Identify signals:

| OA Type | JD Signals |
|---------|-----------|
| **Coding** | "HackerRank", "Codility", "LeetCode", "algorithms", "data structures", "take-home coding" |
| **Video Interview** | "HireVue", "Sonru", "video screening", "async interview", "recorded responses" |
| **Psychometric** | "aptitude", "numerical reasoning", "verbal reasoning", "abstract reasoning", "SHL", "Revelian", "Criteria Corp" |
| **Case Study** | "case study", "business case", "written analysis", "consulting", "strategy" |
| **Work Simulation** | "realistic job preview", "work sample", "situational judgement", "SJT" |
| **Written Assessment** | "written response", "policy brief", "analysis task", "essay" |

If signals are ambiguous, ask: "Do you know what type of OA this is? (coding / video / psychometric / case study / other)"

### Step 2 — Generate Prep Plan

**Coding OA:**
```
🖥️  Coding Assessment Prep — {company} · {role}

Predicted type: {platform if detectable, e.g. HackerRank}
Time limit: typically 60–90 min | 2–3 problems

Focus areas (based on JD):
  {extracted from JD — e.g. "arrays, hashmaps, SQL queries"}

This week's practice plan:
  Day 1–2  Easy problems — warm up on arrays, strings, loops
  Day 3–4  Medium problems — focus on {JD-specific topic}
  Day 5    Timed mock — simulate real conditions (no hints, timer on)

Recommended resources:
  LeetCode: leetcode.com (filter by company if premium)
  NeetCode 150: neetcode.io — curated list, free
  HackerRank practice: hackerrank.com/domains/algorithms

Tips:
  • Talk through your approach before coding
  • Handle edge cases: empty input, null, duplicates
  • Test with examples from the problem before submitting
```

**Video Interview (HireVue / Sonru):**
```
🎥  Video Interview Prep — {company} · {role}

Format: async — you record responses, no live interviewer
Typical structure: 3–5 questions, 30–90 sec prep, 1–3 min response

Common question types:
  • "Tell me about yourself" — 90 sec elevator pitch
  • "Why {company}?" — 2-3 specific reasons
  • "Tell me about a time you..." — STAR format
  • Situational: "What would you do if..." — use action-oriented language

Prep tips:
  • Record yourself once — watch it back, fix filler words
  • Look at the camera, not the screen
  • Good lighting + quiet background
  • Dress as you would for an in-person interview

5 practice questions for this role:
  {generate 5 STAR-style questions based on JD soft skill signals}
```

**Psychometric / Aptitude:**
```
📊  Psychometric Test Prep — {company} · {role}

Predicted platform: {SHL / Revelian / Criteria / unknown}
Sections typically included: numerical · verbal · abstract reasoning

Practice resources (free):
  SHL practice: shldirect.com/en/practice-tests
  Revelian: revelian.com/sample-tests
  JobTestPrep: jobtestprep.com.au (paid, worth it for Big 4 / banks)
  Assessment Day: assessmentday.co.uk/aptitudetests (free samples)

Tips:
  • Speed matters — don't dwell; mark and move
  • Numerical: calculator usually allowed; practise reading charts fast
  • Verbal: read the passage first, then the question
  • Abstract: look for rotation, reflection, number of shapes, colour patterns

Recommended daily practice: 20 min × 5 days before the test
```

**Case Study:**
```
📋  Case Study Prep — {company} · {role}

Format: written analysis, usually 1–3 hours, submitted as PDF or Word doc

Structure your response:
  1. Problem statement (2-3 sentences — what is the core issue?)
  2. Key findings (bullet points — data from the case)
  3. Options considered (2-3, with pros/cons)
  4. Recommendation (clear, justified, with implementation steps)
  5. Risks and mitigations

Tips:
  • Structure first, write second — spend 20% of time on outline
  • Use numbers wherever possible — be specific
  • Show you considered multiple options before recommending
  • Proofread — consulting firms penalise sloppy writing

Practice case: {suggest a free McKinsey / BCG / Deloitte sample case relevant to the industry}
```

**Work Simulation / SJT:**
```
🎯  Situational Judgement Prep — {company} · {role}

Format: scenario-based — pick the best/worst response from options
Measures: judgment, values alignment, professional behaviour

How to approach:
  • Think: "What would an ideal employee at this company do?"
  • Prioritise: safety → stakeholders → task completion → efficiency
  • Avoid: extreme responses, blame, ignoring others

Practice: jobtestprep.com.au/situational-judgement-tests
```

### Step 3 — Deadline Alert

Check tracker.json for the application. If `deadline` is within 7 days:
```
⚠️  OA deadline in {n} days — {date}. Start prep today.
```

If `interview_date` is set, count backwards and flag if < 3 days of prep time remain.

### Step 4 — Update Tracker

After generating prep plan, ask: "Want me to log this in your tracker as OA in progress?"
If yes: update application status to `oa`, set `next_step` to "Complete OA by {deadline}".

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
