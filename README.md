<p align="center">
  <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT"/>
  <img src="https://img.shields.io/badge/Agent--Agnostic-✓-success" alt="Agent Agnostic"/>
  <img src="https://img.shields.io/badge/Plays-12-orange" alt="12 Plays"/>
  <img src="https://img.shields.io/badge/Templates-5-green" alt="5 Templates"/>
  <img src="https://img.shields.io/badge/Stages-6-6c63ff" alt="6 Stages"/>
</p>

<h1 align="center">🔁 AI-Native SDLC Skill</h1>

<p align="center">
  A vendor-neutral agent skill covering all six stages of the AI-native Software Development Lifecycle.<br/>
  Drop it into any AI coding agent — Claude Code, GitHub Copilot, Cursor, Aider, or your own setup.
</p>

---

## What This Is

This is a **skill file** — a structured Markdown package that an AI agent reads to understand how to run an AI-native Software Development Lifecycle (SDLC).

It covers 12 plays across 6 stages:

| Stage | Plays |
|-------|-------|
| **1 · Plan** | Capture intent → `intent.md` |
| **2 · Design** | Requirements & design → `spec.md` |
| **3 · Build** | Plan mode, `AGENT.md`, Skills, Parallel sessions & subagents |
| **4 · Test** | Agent feedback loop, Continuous evals in CI |
| **5 · Deploy** | AI PR review, Hooks as approval gates, CI/CD integration |
| **6 · Maintain** | Closing the loop on metrics → back to `intent.md` |

### Shifting the Bottleneck

![Build Bottleneck Moved](assets/fig-bottleneck-moved.png)

When AI agents take on software generation, the build stage collapses from weeks to minutes — shifting human effort to intent capture, architectural review, and governance.

---

## The Line vs. The Loop

![Traditional Line vs AI-Native Loop](assets/fig-line-vs-loop.png)

Traditional software development operates as a slow linear pipeline where every cycle back is a new release project. AI-native development operates as a continuous high-velocity loop, with humans operating above the loop to direct, evaluate, and govern autonomous agents.

---

## Play Dependency Graph

![Play Dependency Graph](assets/fig-play-dependency-graph.png)

```
No prerequisites (start here):
  Capture intent   AGENT.md   Skills   Feedback loop   Hooks   Plan mode

Needs AGENT.md (+ feedback loop helps):
  Parallel sessions & subagents

Needs AGENT.md + feedback loop:
  Continuous evals

Needs capture intent + skills:
  Requirements & design

Needs evals + subagents (skills help):
  AI PR review

Needs PR review + hooks:
  CI/CD integration

Needs CI/CD + capture intent:
  Closing the loop on metrics
```

Start with any top-row play that addresses your biggest bottleneck.

---

## How to Use It

### With Claude Code / Antigravity

```bash
# Clone into your global skills directory
git clone https://github.com/sakthiveltofficial/ai-native-sdlc \
  ~/.gemini/config/skills/ai-native-sdlc

# Or into a project-level skills directory
git clone https://github.com/sakthiveltofficial/ai-native-sdlc \
  .agents/skills/ai-native-sdlc
```

The skill auto-activates when you ask about `AGENT.md`, `intent.md`, `spec.md`, hooks, evals, PR review loops, or closing the production feedback loop.

### With Any Other Agent

Point your agent's skill / system-prompt loading mechanism at `SKILL.md`. The YAML front matter (`name`, `description`) is the trigger — the body is loaded when it matches.

---

## Structure

```
ai-native-sdlc/
├── SKILL.md                               ← Entry point — load this first
├── assets/
│   ├── fig-bottleneck-moved.png           Visual: Build phase bottleneck diagram
│   ├── fig-line-vs-loop.png               Visual: Traditional vs AI-Native SDLC loop
│   └── fig-play-dependency-graph.png      Visual: 12-play dependency graph
├── references/
│   ├── 01-capture-intent.md               Stage 1 · Plan
│   ├── 02-requirements-and-design.md      Stage 2 · Design
│   ├── 03-plan-mode.md                    Stage 3 · Build
│   ├── 04-agent-md.md                     Stage 3 · Build
│   ├── 05-skills-as-institutional-knowledge.md
│   ├── 06-parallel-sessions-and-subagents.md
│   ├── 07-feedback-loop.md                Stage 4 · Test
│   ├── 08-continuous-evals-in-ci.md
│   ├── 09-ai-pr-review.md                 Stage 5 · Deploy
│   ├── 10-hooks-as-approval-gates.md
│   ├── 11-cicd-integration.md
│   └── 12-closing-the-loop-on-metrics.md  Stage 6 · Maintain
└── examples/
    ├── AGENT.md.template       ← Copy to your repo root and fill in
    ├── intent.md.template
    ├── spec.md.template
    ├── plan.md.template
    └── REVIEW.md.template
```

---

## The Artifact Chain

Every stage ends by committing one artifact. The next stage reads it. The chain of commits is your audit trail.

```
intent.md → spec.md → plan.md → diff + tests → PR + review → incident record
    ↑                                                                  │
    └─────────── production breach writes the next intent.md ─────────┘
```

---

## Generic Terminology

This skill is **agent-agnostic**. Every tool-specific name has a generic equivalent:

| Generic (used in this skill) | Claude Code | GitHub Copilot | Cursor |
|---|---|---|---|
| `AGENT.md` | `CLAUDE.md` | `.github/copilot-instructions.md` | `.cursorrules` |
| Agent settings file | `.claude/settings.json` | — | `.cursor/settings.json` |
| Agent hook | Claude Code hooks | — | Cursor rules |
| Skill file | Claude Code skills | — | System prompt includes |
| Subagent definition | `.claude/agents/*.md` | — | Custom agents |

---

## Quick-Start (10 Steps)

1. Copy [`examples/AGENT.md.template`](examples/AGENT.md.template) → your repo root, fill in, commit
2. Write your first [`intent.md`](examples/intent.md.template) from a real ticket or user pain point
3. Wrap build + test into one command; add it to `AGENT.md`
4. Start a session in plan mode; commit the resulting [`plan.md`](examples/plan.md.template) before code is written
5. Write your first policy as a skill file
6. Add one hook that blocks the most dangerous action the agent could take
7. Add one eval for the last bug that reached production
8. Add the agent as first PR reviewer using [`REVIEW.md`](examples/REVIEW.md.template)
9. Gate the pipeline merge on the agent verdict
10. Wire one production metric back to `intent.md` generation

---

## Governance Principles

- The agent that wrote code cannot approve it — separation of duties is structural
- Every gate is enforced by configuration (hooks, managed settings), not by convention
- The audit trail is the commit history
- Human judgment is reserved for what matters: policy calls, flagged findings, regulated code
- All controls live in Git and are reviewed like code

---

## Contributing

PRs welcome for:
- Improvements to existing plays
- New skill files for specific domains (security, accessibility, performance)
- Agent-specific mapping tables (Gemini CLI, Codeium, Windsurf, etc.)
- Real-world examples and case studies

**Guideline:** keep skills generic-first. If something is agent-specific, document it alongside the generic version in a table.

---

## License

[MIT](LICENSE)

---

## Acknowledgements

Distilled and generalized from the **AI-Native SDLC Playbook** course on [Claude Academy](https://academy.claude.com/courses/ai-native-sdlc-playbook). Course content © Anthropic; this derivative work and all additions are MIT licensed.
