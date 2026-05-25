# JobRadar: App vs. Skill — Strategic Decision

> Date: 2026-05-26 | Source: Multi-agent debate (4 specialist workers)

## TL;DR

**Build as a Claude Skill first. Validate, then consider a standalone app.**
Verdict from 4-agent debate: **3–1 in favour of Skill.**

---

## The Question

Should JobRadar be built as a **standalone app** or as a **Claude AI Skill/Plugin**?

---

## Agent Votes & Rationale

### 📈 Market Analyst → SKILL

- AI recruitment market at $8.16B in 2025, growing at 24.8% CAGR to $15.24B by 2030
- 70%+ of job seekers already use generative AI for applications — no dominant tool exists yet
- Claude Skills ecosystem grew 900% in 3 months, now 1.2M+ skills; distribution infrastructure already built
- Embedded AI is winning: 40% of enterprise software will include task-specific agents by end of 2026 (up from <5% in 2025)
- Job seekers use 3–4 tool stacks, not monoliths — favours a skill that plugs into an existing AI assistant

**Risk:** Platform dependency — Anthropic controls distribution

---

### 🔍 Competitive Intel → SKILL

**Standalone app competitors (brutal market):**
- Teal — $20.7M raised, full career OS
- Rezi — 4.3M users, bootstrapped resume builder
- Simplify — YC-backed, auto-fill browser extension

**Skill ecosystem competitors (nearly empty):**
- A few shallow, unbranded Claude/ChatGPT job skills with no workflow depth
- Claude Skills search term grew 900% in 3 months — first-mover advantage available now

Entering the standalone app market means 18+ months and $2M+ to reach feature parity with incumbents. The skill ecosystem is wide open.

**Risk:** Anthropic could build a first-party job search product

---

### 🧠 Product Strategist → SKILL first, App later

- Skill ships in days; standalone app requires months of infra (auth, billing, mobile/web surfaces, payments)
- Claude's user base = zero CAC — distribution without cold-start
- Iteration velocity is the real moat at this stage: 50 experiments in the time it takes to ship an app MVP
- Monetisation ceiling is real but premature — LTV only matters once you have retention
- **Middle path:** use the Skill as acquisition + discovery layer; capture emails and outcome data; spin out an app once a power-user cohort is visible

**Risk:** Revenue share ceiling vs. owned subscription LTV

---

### 👤 User Advocate → APP (dissent)

- Job search is a 55–71 day campaign — needs dashboards, persistent tracking, daily notifications
- 48% of job seekers mass-apply and need background automation, not chat sessions
- Data sensitivity: users hesitant to share CVs with a general AI assistant; a branded app feels safer
- Habit formation requires a dedicated home with a return loop
- App gates out users who don't have a Claude subscription

**Risk:** App requires significant engineering investment and competes with funded incumbents

---

## Synthesis Verdict

The Skill wins in 2025–2026. The User Advocate's concerns (persistence, habit loops, trust) are real but solvable within the skill:
- Add persistent state via external storage (e.g. GitHub-based scan history already working)
- Capture user emails through the digest flow
- Track outcomes per application
- Build the *JobRadar brand*, not just "a Claude skill"

When a power-user cohort with clear retention patterns emerges, that's the signal to spin out an app — with a real data moat already built.

---

## Agreed Roadmap

| Phase | Timeline | Focus |
|-------|----------|-------|
| 1 | Now → 3 months | Deepen skill: full loop, state persistence, email capture, outcome tracking |
| 2 | 3–9 months | Grow on Claude's user base; track retention and outcome signals |
| 3 | 9–15 months | Light companion dashboard + notifications layer |
| 4 | 15+ months | Full standalone app — only if user data justifies it |

---

## Shared Risk & Mitigation

**Platform dependency on Anthropic is existential for the Skill path.**

Mitigate by:
- Building the *JobRadar* brand identity from day one (not just "a Claude skill")
- Capturing user emails via the Gmail digest flow
- Owning outcome data (applications tracked, interviews landed, offers received)
- Keeping the skill logic portable — prompt + config files that could run on any LLM platform

---

## Current State (as of 2026-05-26)

The skill is already well into Phase 1:
- ✅ 9 slash commands: `/job-scan`, `/job-eval`, `/job-cv`, `/job-cover`, `/job-gaps`, `/job-prep`, `/job-digest`, `/job-track`, `/job-status`
- ✅ Config fully set up for Cynthia (intern, 9 role types, Melbourne/Sydney/Remote, 70+ target companies)
- ✅ Daily scans running (May 22 & 23 complete, 28 jobs found)
- ✅ Gmail digest wired via MCP
- ✅ CV in `profile/DefaultCV-Cynthia.docx`
- ⬜ State persistence / outcome tracking — not yet built
- ⬜ User email capture — not yet built
- ⬜ Brand identity beyond the skill — not yet built

---

## Do Not Relitigate

This decision is made. Future workers on this project should focus on **deepening the skill** and **building brand + data assets** — not restarting the App vs. Skill debate.
