# Stage 3 · Build: AGENT.md — The Agent Context File

## What This Play Does

`AGENT.md` gives every AI agent session the context a new team member would need
on day one: conventions, commands, architecture decisions, and the mistakes the
team has seen most often. Knowledge that used to live in people's heads and team
wikis becomes a file the agent reads at the start of every session — maintained
by the whole team and updated whenever a mistake is made.

> **Naming note:** This file is called `CLAUDE.md` in Claude Code and may be
> named differently in other agents (e.g., `.github/copilot-instructions.md` in
> Copilot, `AGENTS.md` in OpenAI Codex). Use whatever name your toolchain
> reads automatically. The pattern and content are the same regardless of name.

---

## What Changes

| Traditional | AI-native |
|---|---|
| Institutional knowledge lives in people's heads, onboarding docs, and Confluence pages that go stale. Every new engineer and every new agent session starts without it | `AGENT.md` is the living document every session reads first. Corrections go in on the day the mistake is caught, and Git history shows who changed what and why |

---

## Getting Started

- **Prerequisites:** None. This is a top-row play with no dependencies.
- **Infrastructure:** A repository and one engineer who knows the codebase well.

---

## How to Execute It

1. Ask the agent to generate a starting `AGENT.md` by reading the repository
   (many agents have an `/init` or equivalent command for this).
2. Cut the generated file down to what a new joiner would need on day one:
   - Build, test, and lint commands
   - Conventions that matter (language version, forbidden patterns, naming rules)
   - Architecture overview (key directories, data flow, external dependencies)
   - The mistakes the agent keeps making
3. Check `AGENT.md` into Git at the repo root so the whole team shares one
   version and changes are reviewed like code.
4. Apply the correction rule: **when the agent makes a mistake twice, the
   correction goes into `AGENT.md`.**
5. Keep it under one page. The agent reads the whole file at the start of every
   session; anything stale takes up context for no benefit.

---

## `AGENT.md` Template

```markdown
# [Service / repo name]

## Commands
- Build:   <single command to build>
- Test:    <unit test command>   (note: integration tests need <prerequisite>)
- Lint:    <lint command>        (runs in CI; fix before pushing)

## Conventions
- <Language and version>. <Key library choices and what is forbidden.>
- <Domain invariant that must never be violated, e.g., "Money is always Decimal, never float.">
- <Every public endpoint needs an integration test.>

## Architecture
- `<directory>/` holds <what>; `<directory>/` holds <what>.
- <Key external system> events are defined in `<schema path>/`; never edit generated classes.

## Things the agent gets wrong
- Do not bump dependency versions; the platform team owns them.
- The legacy `<path>/` package is frozen; changes go in `<new path>/`.
- <Any repeated mistake your team has caught.>
```

### Concrete Example

```markdown
# Payments service

## Commands
- Build: make build
- Test:  make test (unit), make itest (integration — needs Docker)
- Lint:  make lint (runs in CI; fix before pushing)

## Conventions
- Java 21, Spring Boot 3. No new Lombok.
- Money is always BigDecimal, never double.
- Every endpoint needs an integration test in src/itest.

## Architecture
- api/ holds REST controllers, core/ holds domain logic, adapters/ talks to
  external systems.
- Kafka events are defined in schemas/; never edit generated classes.

## Things the agent gets wrong
- Do not bump dependency versions; the platform team owns them.
- The legacy v1/ package is frozen; changes go in v2/.
```

---

## Governance Considerations

- `AGENT.md` is version controlled, so the instructions the agent works under
  are reviewable and auditable by any team member or auditor.
- Team conventions are applied consistently through the file, not by hoping
  each engineer remembers to brief the agent.
- Code owners can require review approval for changes to `AGENT.md` in the same
  way as other sensitive files.

---

## Relationship to Skills

`AGENT.md` covers what is specific to this repository. Skills (see
[05-skills-as-institutional-knowledge.md](05-skills-as-institutional-knowledge.md))
cover what is shared across repositories — security policies, brand guidelines,
architecture standards. The two are complementary: `AGENT.md` for repo-specific
context, skills for organization-wide policies.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | How often the agent repeats a mistake that `AGENT.md` should have caught (track corrections as commits) | Git history on `AGENT.md` |
| **Lagging** | Time to first merged PR for a new team member | PR history |
