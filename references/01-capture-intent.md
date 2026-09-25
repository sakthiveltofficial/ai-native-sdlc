# Stage 1 · Plan: Capture Intent

## What This Play Does

Replaces ad-hoc requirements gathering — emails, meetings, long ticket comments —
with a single, human-readable and machine-actionable file: **`intent.md`**.

The AI agent synthesizes pain points directly from the sources (support tickets,
user interviews, incident post-mortems, team discussions) and writes up a
structured artifact that the product owner can review and commit. The committed
`intent.md` is the signal that triggers the next stage (Requirements & Design).

---

## What Changes

| Traditional | AI-native |
|---|---|
| Requirements gathered by committee, distilled through workshops and sign-offs, written by hand over days or weeks | Agent synthesizes pain points straight from the sources and captures them in `intent.md`, which is human-readable and machine-actionable — in a single session |

---

## Getting Started

- **Prerequisites:** None. This is a top-row play with no dependencies.
- **Infrastructure:** Access to the source material (tickets, interview notes,
  Slack threads, post-mortems) and an AI agent that can read them.

---

## How to Execute It

1. Collect the raw sources: support tickets, user-research notes, incident
   post-mortems, product feedback, or a recorded conversation with stakeholders.
2. Feed the sources to the agent and ask it to synthesize them into `intent.md`,
   using the schema below.
3. Review `intent.md`. Resolve any open questions with the named policy owners
   before the file is committed.
4. Commit `intent.md` to the repository. The commit itself is the handoff signal
   to Stage 2 (Requirements & Design).
5. When an accepted `intent.md` exists, subsequent stages read it rather than
   reconstructing intent from scratch.

---

## `intent.md` Schema

```markdown
# Intent: [Short title]

## Problem
One paragraph. What is broken or missing, and who feels it.

## Desired outcome
What success looks like from the user's or operator's perspective.
Measurable where possible.

## Affected systems
List the services, repositories, or data stores likely in scope.

## Constraints
Regulatory, contractual, or architectural limits that cannot be negotiated.

## Open questions
Numbered list. Each question names the person or team who can resolve it.

## Source references
Links or IDs to the tickets, interviews, or incidents this synthesizes.
```

---

## Prompt Template

```
Read the attached source material and produce an intent.md using the schema
provided. Highlight every point where sources conflict or where a policy question
must be resolved before engineering can start. Do not resolve the conflicts
yourself — mark them as open questions and name the right owner for each.
```

---

## Governance Considerations

- The product owner reviews and approves `intent.md` before it is committed.
- Open questions must be resolved (or explicitly deferred) by the named owners
  before the file enters the design stage.
- `intent.md` is version controlled, so changes to requirements are visible in
  Git history and reviewable by any stakeholder or auditor.
- The file is also the seed for the feedback loop at Stage 6 (Maintain): when a
  production metric breaches a control band, the agent writes the next
  `intent.md` automatically.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Time from incident/request to `intent.md` committed | Git timestamps |
| **Lagging** | Requirements rework after build starts — count `spec.md` commits dated after the first `plan.md` commit for the same change | `git log` with file filter |
