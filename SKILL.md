---
name: whatsup
description: Render a compact task status panel summarizing the current Claude Code session — goal, progress %, status (working/waiting/blocked/done), next action, and a recent timeline. Use when the user asks "whatsup", "what's up", "status", "status panel", "where are we", "what's left", "summarize progress", "task dashboard", "give me a HUD", or types /whatsup or /status, especially mid-task or when resuming a long session.
---

# Status Panel

Summarize the **current session** into a compact dashboard, modeled on the Codey Task HUD. Everything you need is already in the conversation — do not investigate the codebase. Just read the session and render the panel.

## Output format

Render the panel inside a fenced ``` block as a **fully outlined box** of **fixed width** (`┌─┐ │ ├─┤ └─┘` on all four sides, sections split by `├─┤`). The 10-cell bar is `█` filled / `░` empty (`round(progress/10)` cells filled).

**Fixed width — never grow the box; wrap instead.** The box is a constant **58 columns** wide (total, including both `│`) so it fits a small/narrow terminal. Lines never extend it — long values wrap onto new lines inside the box.

```
┌─ STATUS ───────────────────────────────────────────────┐
│ GOAL    <goal, ≤ 8 words>                              │
│         <brief, wrapped to fit, ≤ 30 words>            │
│         <…brief continues here>                        │
├────────────────────────────────────────────────────────┤
│ STATE   [<status>] <bar> <progress>%   ·   <stepLabel> │
├────────────────────────────────────────────────────────┤
│ NEXT    <next step or open question, ≤ 10 words>       │
│         <one-line elaboration, optional>               │
├────────────────────────────────────────────────────────┤
│ RECENT  • <newest event>                    — <when>   │
│         • <event>                           — <when>   │
│         • <a longer event that needs to wrap onto      │
│           a second line>                    — <when>   │
└────────────────────────────────────────────────────────┘
```

**How to lay it out (do this exactly):**
1. Each row is `│ ` + an 8-char label column (`GOAL`/`STATE`/`NEXT`/`RECENT`, or 8 spaces for continuation/wrapped lines) + the value, then pad with trailing spaces and close ` │`. The inner area is **54 columns**; the value area after the label is **46**.
2. **Wrap, don't widen.** If a value exceeds the value area (46 cols), break it at word boundaries onto continuation lines that reuse the 8-space blank label. For a wrapped timeline bullet, indent the continuation 2 spaces so text lines up under the `•`.
3. The top rule (`┌─ STATUS ─…─┐`), every divider (`├─…─┤`), and the bottom rule (`└─…─┘`) all span the same **58** columns. (Treat each `█ ░ ─ │ •` as one column.)

**Section dividers:** a `├─┤` rule between every section — after GOAL/brief, after STATE, after NEXT — plus the `└─┘` close after RECENT.

**Timeline timestamps:** **right-align** each `— <when>` to the box's right inner edge — because the box is fixed-width, they form a clean vertical column with no manual padding. If an entry's last line leaves no room for the timestamp, put `— <when>` on its own continuation line, still right-aligned.

The label appears once per section; continuation, wrapped, and extra timeline lines use the 8-space blank label. Show the timeline newest-first: up to **8** entries covering the meaningful steps — every notable action, decision, and dropped approach you can ground. `<when>` is relative time (`just now`, `Nm ago`, `Nh ago`, `Nd ago`); omit it for an entry you can't ground. Omit the `NEXT` section when `status` is `done`.

## Fields

- **goal** — the overall task in one line. If unclear, infer from the first user request.
- **brief** — one line expanding on the goal: the scope, the why, or what "done" means. ≤ 30 words. Omit if it would just restate the goal.
- **status** — one of:
  - `waiting` — blocked on a user decision/answer (you asked something and are waiting)
  - `blocked` — stuck on an external problem (failing dep, missing access, broken env)
  - `done` — task finished and verified
  - `working` — anything else (default)
- **progress** — integer 0–100. Determine it **step-based**, not by gut feel:
  1. Break the goal into the discrete subtasks needed to reach "done" (the same steps as `stepLabel`).
  2. `progress = round(100 × completed / total)`. A subtask counts as completed only once it's verified (test passed, file written, command succeeded) — half-done work doesn't count.
  3. Anchors: nothing started = `0`; everything done **and verified** = `100`. Cap at `95` while any step is unfinished, unverified, or `status` is `waiting`/`blocked` — only a fully verified task reaches `100`.

  If the goal has no clean step breakdown, fall back to a coarse estimate (`25` / `50` / `75`) and say why in the brief — don't invent false precision like `78%`.
- **stepLabel** — optional, e.g. `step 3 / 5` when the work has clear discrete steps.
- **nextAction** — the single most useful next step, or the open question if `waiting`.
- **timeline** — concrete things that happened (`progress`, `action`, `decision`, or `dropped`/abandoned approach). Newest first, one per line under the `RECENT` label. `when` = relative time (`just now`, `Nm ago`, `Nh ago`, `Nd ago`).

## Rules

- **Terse.** Hard caps: goal ≤ 8 words, nextAction ≤ 10 words, each timeline entry ≤ 14 words. Timeline entries may use the extra room to name files, symbols, or the reason — still fragments, not full sentences. No trailing punctuation.
- **English**, regardless of the conversation's language.
- **Ground it** in what actually happened this session — never invent progress, events, or timestamps you can't justify.
- **Be honest about status.** If you just asked the user something and haven't acted, that's `waiting`. If tests are failing and unresolved, that's `working` (or `blocked` if external), not `done`.
- Render only the panel — no preamble or explanation around it unless the user asks.

## Example

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
