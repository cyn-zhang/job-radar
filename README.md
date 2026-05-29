# JobRadar

**Find better jobs, faster.** JobRadar scans Seek, LinkedIn, GradConnection, and company career pages daily — then helps you apply: honest fit scoring, tailored CVs, cover letters, interview prep, and a Gmail digest. All in Claude.

Works for any level (intern → executive), any industry, any location. Australian focus by default, fully configurable.

---

## What it does

| | |
|---|---|
| 🔍 Daily job scan | Seek, LinkedIn, GradConnection, Indeed + company career pages |
| 🧠 JD evaluation | Honest fit scoring, ATS keywords, Bottom Line verdict |
| 📄 CV tailoring | Strongest evidence mapped to JD, no fabrication |
| 🧩 Gap analysis | Fixable gaps with CV lines; hard gaps with interview scripts |
| ✉️ Cover letter | 4 paragraphs, role-specific, never generic |
| 🎤 Interview prep | STAR behavioural + technical questions, mock interview mode |
| 📬 Gmail digest | Daily email with top picks and full results table |
| 📊 Application tracker | Track status from applied → offer |
| 🗂️ Status dashboard | Active applications, deadlines, closing-soon alerts |

---

## Install

### Claude Code (recommended)

```bash
git clone https://github.com/cynthiazhang/job-radar && cd job-radar
cp config.example.yaml config.yaml   # then edit with your details
claude .
```

Type `/job-scan` or just say "find me jobs" to start.

**Global install** — commands available in any project:
```bash
git clone https://github.com/cynthiazhang/job-radar ~/.claude/skills/job-radar
```

### claude.ai / Claude Desktop / Cowork

1. Download [`SKILL.md`](https://github.com/cynthiazhang/job-radar/blob/main/SKILL.md) from this repo
2. Go to **Customize → Skills → Upload skill** and upload the file
3. Say `/job-setup` to create your config interactively — takes 2 minutes
4. Say "find me jobs" to run your first scan

---

## Config

Run `/job-setup` in chat to create your `config.yaml` interactively. Or copy `config.example.yaml` and edit directly.

Key fields:

| Field | Example |
|-------|---------|
| `hunter.name` | Your name |
| `hunter.email` | Gmail digest destination |
| `hunter.level` | `intern` / `graduate` / `junior` / `mid` / `senior` / `lead` / `manager` / `director` |
| `hunter.roles` | `["Software Engineer Intern", "Data Engineer Intern"]` |
| `hunter.locations` | `["Melbourne", "Sydney", "Remote"]` |
| `hunter.industry` | `["tech"]` — or `["any"]` for no filter |
| `hunter.work_type` | `internship` / `graduate` / `contract` / `permanent` / `any` |
| `hunter.target_companies` | Companies to check career pages directly |

Update any field in chat: "add Brisbane to my locations", "change my level to graduate".

---

## Commands

| Command | What it does |
|---------|-------------|
| `/job-scan` | Scan all sources → save results, update dedup index |
| `/job-eval` | Evaluate a JD (paste or link) |
| `/job-cv` | Tailor CV for a role |
| `/job-gaps` | Coverage map + gap analysis + recruiter view |
| `/job-cover` | Write a cover letter |
| `/job-prep` | Interview prep + mock interview |
| `/job-digest` | Scan + send Gmail digest |
| `/job-track` | Add or update an application |
| `/job-status` | Dashboard of all active applications |
| `/job-setup` | Create or update your config |

Or just talk naturally — "find me jobs", "evaluate this JD", "I applied to Atlassian".

**Auto-chain:** paste a JD → analysis → gap map → CV tailoring → cover letter → track application.

---

## Gmail digest

Connect Gmail via **Customize → Connectors → Gmail** in claude.ai or Claude Desktop. Digests land in your Drafts — open and send when ready.

**Cron (Claude Code):**
```bash
0 23 * * * cd /path/to/job-radar && claude -p "run daily job scan and send Gmail digest" --model claude-haiku-4-5-20251001
```
Adjust time for your timezone (23:00 UTC = 9:00 AEST).

---

## File structure

```
job-radar/
├── SKILL.md                  ← skill definition
├── config.example.yaml       ← config template
├── config.yaml               ← your settings (gitignored)
├── plugin.json               ← Claude marketplace manifest
├── .claude/commands/         ← slash commands
├── references/               ← platform tips, ATS taxonomy
├── applications/             ← your job applications (gitignored)
│   ├── tracker.json          ← application tracker (source of truth)
│   └── {Company}/{Role}/     ← per-application files
├── scans/                    ← daily scan results (gitignored)
├── seen.json                 ← scan dedup index (gitignored)
└── profile/                  ← your CV (gitignored)
```

---

## Principles

- **Never fabricate** — honest gaps with interview scripts beat inflated CVs
- **ATS-first** — exact JD phrases in every CV and cover letter
- **Bottom Line always** — every JD evaluation ends with fit score + verdict + dealbreaker
- **Flag deadlines** — 🔴 anything closing within 7 days
