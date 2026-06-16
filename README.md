# whatsup

An agent skill that renders the current session as a compact, fixed-width **status panel** — goal, progress, status, next action, and a recent timeline — inspired by the [Codey](https://github.com/its-ahoh/codey/) Mac app's Task HUD. Works with Claude Code and any AI coding agent that supports skills.

```
┌─ STATUS ───────────────────────────────────────────────┐
│ GOAL    Wire up auth token refresh                     │
│         silently renew expired tokens so users stay    │
│         logged in across restarts                      │
├────────────────────────────────────────────────────────┤
│ STATE   [working] ████████░░ 78%   ·   step 4 / 5      │
├────────────────────────────────────────────────────────┤
│ NEXT    Add retry on 401 responses                     │
│         wrap fetch in interceptor                      │
├────────────────────────────────────────────────────────┤
│ RECENT  • refresh endpoint added            — 2m ago   │
│         • token store wired                 — 8m ago   │
│         • added interceptor with retry and back-       │
│           off on expired tokens            — 12m ago   │
└────────────────────────────────────────────────────────┘
```

## Install

Clone into your agent's skills directory. For Claude Code:

```bash
git clone https://github.com/its-ahoh/whatsup.git ~/.claude/skills/whatsup
```

For other agents, clone into wherever that agent loads skills from.

## Usage

Invoke it mid-task to get a snapshot of where things stand:

- `/whatsup`
- or just ask: "whatsup", "what's up", "where are we", "summarize progress"

The panel is a fixed **58-column** outlined box that wraps long lines so it fits a narrow terminal, with right-aligned timestamps that form a clean column. See [`SKILL.md`](SKILL.md) for the full format spec.
