# Stage 2 · Design: Requirements and Design

## What This Play Does

Compresses the traditional requirements-then-design two-phase handoff into a
single working session. The agent reads the committed `intent.md`, applies the
team's encoded skills (security policies, architecture standards, brand
guidelines), and produces a `spec.md` — a single document that is both the
requirements record and the design decision record.

---

## What Changes

| Traditional | AI-native |
|---|---|
| Spec written by analysts, parsed by designers — separate documents, separate reviews, days or weeks apart | Requirements and design compressed into one session with the agent, guided by skills that encode standards, versioned in Git alongside the intent |

---

## Getting Started

- **Prerequisites:** A committed `intent.md` (Stage 1). Skills that encode your
  policies and standards help significantly (Stage 3 – Skills).
- **Infrastructure:** An AI agent with read access to the repository and the
  team's skill files.

---

## How to Execute It

1. Open a fresh agent session. Give it the `intent.md` and ask for a
   requirements and design spec as `spec.md`.
2. Instruct the agent to apply any available skills — security, architecture,
   brand, UX — as constraints on the spec.
3. Ask the agent to explicitly flag every area of concern, especially where
   policy conflicts arise or where the intent cannot be satisfied by existing
   standards.
4. Work through the flagged concerns first. Each concern is a point an analyst
   would have escalated. The product owner resolves each one with the relevant
   policy owner before engineering sees the spec.
5. Commit `spec.md` alongside `intent.md`. The file pair records what was asked
   for and what was decided.
6. The product owner decides — with a technical lead for higher-risk changes —
   whether the spec and intent move forward to Stage 3 (Build). A human always
   makes this call.

---

## `spec.md` Schema

```markdown
# Spec: [Title from intent.md]

## Intent reference
Link or commit SHA of the intent.md this spec derives from.

## Requirements
Numbered list. Each requirement is traceable to a source in intent.md.

## Design decisions
For each significant decision: the options considered, the choice, and the reason.

## Constraints applied
List the skills / policies applied and how they shaped the design.
Reference the skill name and version.

## Areas of concern
Numbered list. For each: the conflict or gap, the owner, and the resolution or
deferral decision.

## Out of scope
Explicit list of what this change does NOT address, to bound the build.

## Acceptance criteria
Testable statements. These become the basis for the eval suite (Stage 4).
```

---

## Prompt Template

```
Read the attached intent.md and produce a requirements and design spec as spec.md.
Apply the skills available to you so the plan conforms to our architecture
guidelines, security policies, and UX standards. Document the spec fully, ready
to hand to the engineering team. Describe clearly any areas of concern, especially
where you cannot satisfy contradicting policies.
```

---

## Legacy System Integration

If your organization uses Jira, ServiceNow, or a regulated requirements tool,
pick one source of truth per artifact. Three viable configurations:

| Configuration | When to use |
|---|---|
| **Repo is truth** — Markdown artifacts are authoritative; legacy system references commits | Engineering-led orgs where all records live in one tool |
| **Legacy system is truth** — Jira/ServiceNow is authoritative; agent reads the record at session start and writes outcome back via MCP | Regulated orgs where auditors already accept the existing tool |
| **Linkage minimum** — all artifacts note the record ID; all legacy records contain the commit SHA | Good starting point when transitioning — two sources of truth with explicit links |

---

## Governance Considerations

- Policy is applied while the spec is written, not discovered in a review weeks
  later. The skills in force at the time of writing are logged in version control.
- The product owner signs off on the spec and routes flagged concerns to named
  policy owners.
- The spec, the prompt that produced it, and the skill versions used are all in
  version control — the record is complete without any separate documentation step.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Elapsed time between `intent.md` commit and `spec.md` commit | Two Git timestamps on the same feature branch |
| **Lagging** | Requirements rework after build starts — `spec.md` commits dated after the first `plan.md` commit | `git log --follow` |
