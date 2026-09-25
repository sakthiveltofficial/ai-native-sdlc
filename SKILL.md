---
name: ai-native-sdlc
description: >
  End-to-end playbook for running an AI-native Software Development Lifecycle (SDLC).
  Covers all six stages — Plan, Design, Build, Test, Deploy, Maintain — with concrete
  plays, artifact conventions, governance patterns, and measurement guidance.
  Generic: works with any AI coding agent (Claude Code, Copilot, Cursor, Aider, etc.).
  Activate when the user asks about AI-native SDLC, agentic engineering workflows,
  AGENT.md, intent.md, spec.md, plan.md, hooks as gates, evals in CI, or
  closing the feedback loop on production metrics.
---

# AI-Native SDLC Playbook

A practical, vendor-neutral playbook for transforming your software development
lifecycle so that AI agents are first-class contributors at every stage — while
humans remain accountable for every decision that requires judgment.

> **Read the reference files in `references/` for full detail on any play.**
> Start here for the quick map, then follow links to what you need.

---

## Why Transform the SDLC?

When AI agents write code faster than traditional processes allow, three things
break:

1. **The bottleneck moves.** Build collapses to agent speed. Plan, Review, and
   Deploy still run at human speed and become the new constraint.
2. **Controls stop matching reality.** Line-by-line human review was sized for
   human output. It cannot keep up once agents produce most of the diff.
3. **Governance costs spike.** Exceptions still route through committees that
   meet weekly or monthly while agents produce output daily.

The AI-native SDLC replaces the traditional linear pipeline with a **continuous
loop** — each stage ends by committing an artifact that triggers the next stage,
with humans concentrating their attention at the gates.

---

## The Six Stages and Their Plays

| Stage | Play | Key Artifact | Reference |
|-------|------|-------------|-----------|
| **1 · Plan** | Capture intent | `intent.md` | [01-capture-intent.md](references/01-capture-intent.md) |
| **2 · Design** | Requirements & design | `spec.md` | [02-requirements-and-design.md](references/02-requirements-and-design.md) |
| **3 · Build** | Agent plan mode | `plan.md` | [03-plan-mode.md](references/03-plan-mode.md) |
| **3 · Build** | AGENT.md | `AGENT.md` | [04-agent-md.md](references/04-agent-md.md) |
| **3 · Build** | Skills as institutional knowledge | skill files | [05-skills-as-institutional-knowledge.md](references/05-skills-as-institutional-knowledge.md) |
| **3 · Build** | Parallel sessions & subagents | agent definitions | [06-parallel-sessions-and-subagents.md](references/06-parallel-sessions-and-subagents.md) |
| **4 · Test** | Give the agent a feedback loop | verification block | [07-feedback-loop.md](references/07-feedback-loop.md) |
| **4 · Test** | Continuous evals in CI | eval suite | [08-continuous-evals-in-ci.md](references/08-continuous-evals-in-ci.md) |
| **5 · Deploy** | AI in the PR review loop | `REVIEW.md` | [09-ai-pr-review.md](references/09-ai-pr-review.md) |
| **5 · Deploy** | Hooks as approval gates | hook scripts | [10-hooks-as-approval-gates.md](references/10-hooks-as-approval-gates.md) |
| **5 · Deploy** | CI/CD integration | pipeline config | [11-cicd-integration.md](references/11-cicd-integration.md) |
| **6 · Maintain** | Closing the loop on metrics | `bands.yaml` + `intent.md` | [12-closing-the-loop-on-metrics.md](references/12-closing-the-loop-on-metrics.md) |

---

## Play Dependency Graph

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

Plays in the top row have no prerequisites. A solid dependency means required;
a dotted dependency means it helps but is not required. **Start with any top-row
play that addresses your biggest bottleneck.**

---

## The Artifact Chain

Each stage ends by **committing one artifact**. The next stage begins by reading
it. The chain of commits is also the audit trail.

```
intent.md  →  spec.md  →  plan.md  →  diff + tests  →  PR + review  →  incident record
    ↑                                                                          │
    └──────────────── production breach writes next intent.md ────────────────┘
```

For early stages, Markdown files are the dominant artifact because both a product
owner and an agent can read and act on the same file. From Build onward, the
artifact is code and its records.

---

## Generic Terminology

This playbook uses vendor-neutral terms. Mapping to common tools:

| Generic term | Example tools |
|---|---|
| AI coding agent | Claude Code, GitHub Copilot, Cursor, Aider, Codeium |
| `AGENT.md` | `CLAUDE.md` (Claude Code), `.github/copilot-instructions.md` |
| Agent settings file | `.agent/settings.json`, `.claude/settings.json` |
| Agent hook | Claude Code hooks, Cursor rules, pre-tool scripts |
| Skill file | Claude Code skills, Cursor rules, system prompt includes |
| Subagent definition | `.agents/agents/*.md`, `.claude/agents/*.md` |
| Managed settings | Admin console, MDM-deployed config |
| MCP server | Model Context Protocol server (tool connector) |

---

## Quick-Start Order for a New Team

1. **`AGENT.md`** — give every session the context a new joiner needs.
2. **Capture intent** — replace ad-hoc ticket descriptions with `intent.md`.
3. **Feedback loop** — wrap tests and build into one command the agent can run.
4. **Plan mode** — make the agent write a `plan.md` before touching code.
5. **Skills** — encode your first policy (security, brand, architecture) as a skill.
6. **Hooks** — block the one action that would be catastrophic if the agent did it.
7. **Evals in CI** — add one eval per bug that escapes to production.
8. **PR review** — add the agent as a reviewer once evals catch regressions.
9. **CI/CD** — gate deployment on the agent's verdict once you trust the review.
10. **Closing the loop** — wire production metrics back to `intent.md` generation.

---

## Governance Principles

- **Separation of duties**: the agent that wrote code cannot approve it.
- **Every gate is enforced by configuration**, not by convention. Hooks and
  managed settings run for everyone, every time.
- **The audit trail is the commit history**: who asked for what, what the agent
  produced, and who approved it.
- **Human judgment is reserved for what matters**: policy calls, flagged concerns,
  regulated code, and production incidents.
- **Controls are version controlled**: `AGENT.md`, skills, hooks, `REVIEW.md`,
  and `bands.yaml` all live in Git and are reviewed like code.

---

## Measuring Progress

Each play defines its own leading and lagging indicators. The common signals
to track at the programme level:

| Signal | Source |
|---|---|
| Cycle time (intent → merged PR) | Git timestamps |
| First-pass CI success rate | CI system |
| Review time per PR | PR metadata |
| Requirements rework after build starts | `spec.md` commits after first `plan.md` |
| Defects escaping to production | Incident tracker vs. PR history |
| Time from production breach to `intent.md` | Monitoring log + Git |

---

*Full detail for each play lives in the `references/` directory.*
