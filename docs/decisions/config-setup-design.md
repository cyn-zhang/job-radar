# Config Setup Design

## Problem

When job-radar is published as a skill/plugin, new users need a way to create
their own config.yaml without editing YAML by hand or running a bash script.

## Solution

A `/job-setup` Claude command — a conversational wizard that:
1. Detects whether config.yaml already exists
2. Asks structured questions in plain language
3. Writes config.yaml automatically
4. Works on any platform (Claude Code, Cowork, Claude.ai)

## Why not bash setup script?

The existing `setup` bash script works well for Claude Code power users
cloning the repo directly. But it won't run in:
- Claude.ai web interface
- Cowork mode
- Skill zip installations on any platform

The Claude command approach is universal — it's just a prompt Claude follows.

## Approach chosen

`/job-setup` slash command in `.claude/commands/job-setup.md`
