---
name: "source-command-job-setup"
description: "First-time setup — create your personal config.yaml interactively"
---

# source-command-job-setup

Use this skill when the user asks to run the migrated source command `job-setup`.

## Command Template

You are running the JobRadar setup wizard. Guide the user through creating their `config.yaml` step by step. Be conversational, friendly, and concise. Show one section at a time — never dump all questions at once.

---

## Step 0 — Check for existing config

Check if `config.yaml` exists in the current directory.

If it exists:
```
✅ config.yaml already exists.

Current settings:
  Name:      {hunter.name}
  Level:     {hunter.level}
  Roles:     {hunter.roles joined by ", "}
  Locations: {hunter.locations joined by ", "}
  Industry:  {hunter.industry joined by ", "}

Want to:
  1) Keep it as-is and start scanning → /job-scan
  2) Edit specific settings → tell me what to change
  3) Reset and redo setup → say "reset config"
```
Stop here and wait for the user's choice.

---

## Step 1 — Welcome

If no config.yaml exists, greet the user:

```
👋 Welcome to JobRadar! Let's set up your personal config — takes about 2 minutes.

First, what's your name?
```

Wait for their answer. Save as `{name}`.

---

## Step 2 — Email

```
What email should we send your daily job digest to?
```

Wait. Save as `{email}`.

---

## Step 3 — Career level

```
What stage are you at?

  intern      → student / internship programs
  graduate    → grad programs, < 1 yr experience
  junior      → 1–3 yrs
  mid         → 3–6 yrs
  senior      → 6–10 yrs
  lead        → tech lead / principal
  manager     → engineering manager
  director+   → director, VP, executive
```

Wait. Map their answer to one of: `intern | graduate | junior | mid | senior | lead | manager | director | vp | executive`. Save as `{level}`.

---

## Step 4 — Target roles

```
What roles are you looking for? List a few titles — I'll use these to search job boards.

Examples:
  Software Engineer Intern, Data Engineer Intern, AI Engineer Intern
  Graduate Developer, Junior Backend Engineer
  Product Designer, UX Researcher
```

Wait. Parse their answer into a list of role titles. Save as `{roles}`.

---

## Step 5 — Locations

```
Which cities or regions? (Australian focus by default)

Examples: Melbourne, Sydney, Remote — or just "anywhere"
```

Wait. Parse into a list. "anywhere" → `["Melbourne", "Sydney", "Brisbane", "Perth", "Canberra", "Remote"]`. Save as `{locations}`.

---

## Step 6 — Industry

```
Which industries? I'll focus scans on these sectors.

  tech          → Software, Data, AI, Cybersecurity
  finance       → Banking, Superannuation, Investment
  accounting    → Big 4 / mid-tier accounting firms
  consulting    → Management consulting, strategy
  healthcare    → Medical, pharmaceutical, aged care
  government    → Public sector, defence, policy
  education     → Universities, training, research
  energy        → Mining, utilities, renewables
  retail        → E-commerce, FMCG, consumer
  any           → no filter — search everything

Pick one or more, or say "any".
```

Wait. Parse into a list. Save as `{industry}`.

---

## Step 7 — Work type

```
What type of role?

  internship   → student placement programs
  graduate     → grad programs
  contract     → fixed-term contracts
  permanent    → full-time permanent
  any          → no filter
```

Wait. Save as `{work_type}`.

---

## Step 8 — CV path (optional)

```
Drop your CV into the profile/ folder if you have one.
What's the filename? (press Enter to skip for now)

Example: profile/MyCVName.docx
```

Wait. If they skip, default to `profile/CV.docx`. Save as `{cv_path}`.

---

## Step 9 — Target companies (optional)

```
Any specific companies you'd love to work at? I'll check their career pages directly.

List them, or press Enter and I'll pull the full curated list for your industries.
```

Wait. Two cases:

**If the user lists companies:** use exactly those as `{target_companies_block}` in YAML list format.

**If the user skips:**
- Read `config.example.yaml`
- The `target_companies` block is organised into sections by industry via inline comments (e.g. `# ── Australian Tech`, `# ── Finance & Banking`)
- Include ONLY the sections matching the user's `{industry}` selections, using this mapping:
  - `tech`          → Australian Tech + Global Tech (AU offices) + Quantitative Trading
  - `finance`       → Finance & Banking + Quantitative Trading
  - `accounting`    → Accounting
  - `consulting`    → Consulting
  - `healthcare`    → Healthcare
  - `government`    → Government
  - `legal`         → Legal
  - `education`     → Education
  - `energy`        → Energy / Mining / Resources
  - `retail`        → Retail / E-commerce
  - `marketing`     → Marketing / Media
  - `hr`            → HR / Recruitment
  - `construction`  → Construction / Property
  - `manufacturing` → Manufacturing / Transport / Logistics
  - `any`           → include ALL sections
- Copy only the **uncommented** company lines (lines starting with `    - `) from matching sections
- Preserve the section heading comments (e.g. `    # ── Australian Tech`) and inline timing hints (e.g. `# Feb–Apr closes early ⚠️`)
- Skip lines where the company itself is commented out (starting with `    # - `)
- Save result as `{target_companies_block}` — a ready-to-paste YAML block

---

## Step 10 — Write config.yaml

Once all answers are collected, write `config.yaml` with this exact structure:

```yaml
# ============================================================
# JobRadar — Personal Configuration
# Generated by /job-setup on {today's date}
# Edit anytime — re-run /job-setup to reset.
# ============================================================

hunter:
  name: "{name}"
  email: "{email}"

  level: "{level}"

  roles:
{each role as "    - \"{role}\""}

  eligible_majors: []   # Add degree keywords to filter by eligibility, or leave empty

  locations:
{each location as "    - \"{location}\""}

  industry:
{each industry as "    - \"{industry}\""}

  work_type: "{work_type}"

  sources:
    seek: true
    linkedin: true
    gradconnection: true
    aus_internship_finder: true
    indeed: true
    company_sites: true

  target_companies:
{target_companies_block}
  # Add or remove companies anytime — see config.example.yaml for the full curated list.

  exclude_companies: []
  # exclude_companies:
  #   - "Company You Already Applied To"

  base_path: "applications/"
  cv_path: "{cv_path}"

  cv_format: "docx"
  cover_letter_format: "docx"

  digest_time: "09:00"
  digest_channel: "gmail"
```

After writing, confirm and immediately offer the first action:

```
✅ config.yaml created for {name}!

Your setup:
  Level:     {level}
  Roles:     {roles joined by ", "}
  Locations: {locations joined by ", "}
  Industry:  {industry joined by ", "}
  Digest:    {email} at 09:00

🚀 Ready to find your first jobs?

  1) Run a scan now  → I'll search all your job boards right now
  2) Do it later     → type /job-scan whenever you're ready
```

Wait for their choice.

- If **1 / "yes" / "run scan" / "now"**: immediately run Module 1 (job scan) without asking any further questions. Use config.yaml for all parameters.
- If **2 / "later" / "no"**: respond with one line — `Got it. Type /job-scan whenever you're ready. Good luck! 🎯` — and stop.

---

## Editing later

If the user says "change my [field]" or "update my [field]" at any point in any session, read config.yaml, update only that field, and rewrite the file. Confirm the change with a one-line summary. Never ask them to edit YAML directly.
