# job-radar

**JobRadar** is an AI-powered job hunting copilot. Scans job boards daily, evaluates JDs honestly, maps your CV against requirements, tailors your CV and cover letter, preps you for interviews, tracks your applications, and sends a Gmail digest.

Works for any career level (intern → executive), any industry, any location.

---

## What it does

| | |
|---|---|
| 🔍 Daily job scan | Major job boards and company career pages |
| 🧠 JD evaluation | Honest fit scoring, ATS keywords, Bottom Line verdict |
| 📄 CV tailoring | Strongest evidence mapped to JD, no fabrication |
| 🧩 Gap analysis | Fixable gaps with CV lines; hard gaps with interview scripts |
| ✉️ Cover letter | 4 paragraphs, role-specific, never generic |
| 🎤 Interview prep | Behavioural (STAR) + technical questions, mock interview mode |
| 📬 Gmail digest | Daily email with top picks and full results table |
| 📊 Application tracker | Track status from applied → offer; auto-updated from chat |
| 🗂️ Status dashboard | Summary of all active applications, offers, and next actions |
| ⚙️ Config update | Change roles, locations, level, or companies in chat |

---

## Quick start

JobRadar works in three modes — pick whichever fits your setup:

| Mode | Best for | Features |
|------|----------|----------|
| **Skill** | claude.ai, Claude Desktop, Codex — no terminal | All modules, chat-only output |
| **Project** | claude.ai, Claude Desktop — no terminal | All modules, config embedded in instructions |
| **Claude Code** | Terminal users | All modules + slash commands + file saving + cron |

---

### As a Skill — claude.ai, Claude Desktop, Codex

**Option A — Upload skill file** *(simplest)*

1. Download `templates/job-radar-project.md` from this repo
2. Go to **Customize → Skills → Upload skill** and upload the file
3. Edit the config block (name, email, roles, locations) when prompted
4. Start chatting — "find me jobs", "evaluate this JD"

**Option B — Upload skill zip** *(includes file saving)*

1. Download [`job-radar-skill.zip`](https://github.com/cyn-zhang/job-radar/releases) (or run `./setup --zip` if you have a terminal)
2. Go to **Customize → Skills → Upload skill** and upload the zip
3. Edit your config (name, email, roles, locations) when prompted
4. Start chatting — "find me jobs", "evaluate this JD"

| Platform | Where to upload |
|----------|----------------|
| **claude.ai** | Profile → Customize → Skills → Upload |
| **Claude Desktop** | Customize → Skills → Upload |
| **Codex** | Customize → Skills → Upload |

---

### As a Project — claude.ai, Claude Desktop

1. Open `templates/job-radar-project.md` from this repo
2. Edit the config block at the top (name, email, roles, locations)
3. Paste the whole file into **Project Instructions**
4. Start chatting — "find me jobs", "evaluate this JD"

| Platform | Where to paste |
|----------|---------------|
| **claude.ai** | Projects → New Project → Set project instructions |
| **Claude Desktop** | Projects → New Project → Set project instructions |

---

### As a Claude Code skill — Terminal

**Standalone (recommended)** — a dedicated job-hunting workspace:

```bash
git clone https://github.com/cyn-zhang/job-radar && cd job-radar && ./setup
```

```bash
# open in Claude Code and use any job commands or send message
claude .
/job-digest # or message like 'send me daily job digest email'
```

**Global install** — `/job-*` commands available in every Claude Code project:
```bash
git clone https://github.com/cyn-zhang/job-radar ~/.claude/skills/job-radar
~/.claude/skills/job-radar/setup --global
```

> Want commands in one specific project only?
> ```bash
> git clone https://github.com/cyn-zhang/job-radar ~/.claude/skills/job-radar
> ~/.claude/skills/job-radar/setup --install ~/path-to/my-project
> ```

**Package for sharing** — creates `job-radar-skill.zip`
```bash
cd job-radar && ./setup --zip
```

---

## Commands (Claude Code)

Type `/` in the prompt to see all commands:

| Command | What it does |
|---------|-------------|
| `/job-scan` | Daily scan → saves `scans/Jobs_YYYY-MM-DD.md` |
| `/job-digest` | Scan + Gmail draft to inbox |
| `/job-eval` | Evaluate a JD (paste or link after command) |
| `/job-gaps` | Coverage map + gap analysis + recruiter review |
| `/job-cv` | Tailor CV for a role |
| `/job-cover` | Write a cover letter |
| `/job-prep` | Interview prep + mock interview |
| `/job-track` | Add or update an application in the tracker |
| `/job-status` | Dashboard of all active applications |

Or just talk naturally — "find me jobs", "evaluate this JD", "tailor my CV for Atlassian", "I applied to Google".

**Auto-chain:** paste a JD → analysis → offers gap map → offers CV tailoring → offers cover letter → offer to track application.

---

## Configuration

All settings in `config.yaml` (gitignored, created by `./setup`). Template: `config.example.yaml`.

**Config lookup order** — the skill checks these paths in order and uses the first one found:
1. `./config.yaml` — current working directory (project-local, takes priority)
2. `~/.claude/skills/job-radar/config.yaml` — global fallback

This means you can keep your config in your project directory and all `/job-*` commands will pick it up automatically, without copying or symlinking.

| Field | Description |
|-------|-------------|
| `hunter.name` | Your name |
| `hunter.email` | Gmail digest destination |
| `hunter.level` | `intern` / `graduate` / `junior` / `mid` / `senior` / `lead` / `manager` / `director` / `vp` / `executive` |
| `hunter.roles` | Target role titles to search |
| `hunter.locations` | Cities (e.g. Melbourne, Sydney, Remote) |
| `hunter.eligible_majors` | Degree filter — leave `[]` to skip |
| `hunter.industry` | `tech` / `finance` / `accounting` / `consulting` / `healthcare` / `government` / `legal` / `education` / `energy` / `retail` / `marketing` / `hr` / `construction` / `manufacturing` / `any` |
| `hunter.work_type` | `internship` / `graduate` / `contract` / `permanent` / `any` |
| `hunter.sources` | Toggle each job board on/off |
| `hunter.target_companies` | Companies to check career pages directly |
| `hunter.exclude_companies` | Companies to skip in scans (already applied, not interested) |
| `hunter.cv_path` | Path to your CV (e.g. `profile/YourCV.docx`) |

Say "update my config", "add Brisbane to my locations", or "change my level to graduate" to update settings in chat.

---

## Gmail digest

Uses Gmail MCP to create digest drafts. To connect:

**claude.ai / Claude Desktop**
1. Go to **Settings → Integrations** → find Gmail → **Connect**
2. Authenticate when prompted

**Claude Code (terminal)**
1. Run `/mcp` in the prompt → find Gmail → connect
2. Authenticate when prompted

Digests land in Gmail Drafts — open and send when ready.

**Option 1 — Local cron:**
```bash
crontab -e
# Add (update path, 9am AEST):
0 23 * * * cd /path/to/job-radar && claude -p "run daily job scan and send Gmail digest" --model claude-haiku-4-5-20251001
```

**Option 2 — GitHub Actions (no local machine needed):**

1. Copy `templates/daily-digest.yml` into your own project repo at `.github/workflows/daily-digest.yml`
2. Push to GitHub
3. Add your Anthropic API key: **Settings → Secrets → Actions → New secret** → `ANTHROPIC_API_KEY`
4. The workflow runs at 9am AEST automatically

Scan results are committed back to your repo each day. Adjust the cron time in the workflow file for your timezone.

---

## Optional: gstack (better scraping)

Some job boards block basic web fetching. [gstack](https://github.com/garrytan/gstack) adds headless Chromium for full coverage.

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

JobRadar detects it automatically and uses `/browse` for sites that need it.

---

## File structure

```
job-radar/
├── SKILL.md                  ← skill definition (Claude Code reads this)
├── CLAUDE.md                 ← Claude Code project context
├── config.example.yaml       ← config template
├── config.yaml               ← your settings (gitignored, created by ./setup)
├── setup                     ← install script
├── templates/
│   └── job-radar-project.md  ← paste into claude.ai / Claude desktop / Codex
├── references/               ← platform tips, ATS taxonomy, timing guides
├── applications/             ← gitignored — your job applications
├── scans/                    ← gitignored — daily scan results
└── profile/                  ← gitignored — your CV
```

---

## Principles

- **Never fabricate** — honest gaps with interview scripts beat inflated CVs
- **ATS-first** — exact JD phrases in every CV and cover letter
- **Bottom Line always** — every JD evaluation ends with fit score + verdict + dealbreaker
- **Flag deadlines** — 🔴 anything closing within 7 days
