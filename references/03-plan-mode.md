# Stage 3 · Build: Agent Plan Mode

## What This Play Does

Engineers start every agentic coding session in **plan mode** — a read-only phase
where the agent reads the codebase and produces a written implementation plan
before making any changes. The engineer corrects the plan, then commits it as
`plan.md`. Only then does the agent implement.

This front-loads design review to the cheapest moment: when correcting course
is still a matter of editing a document.

---

## What Changes

| Traditional | AI-native |
|---|---|
| An engineer reads the design and starts writing code. How the change will be made stays in the engineer's head — the first thing a reviewer sees is the finished diff, and by then rework is slow | Work starts with a written plan the agent produces in plan mode, reading the codebase without changing anything. The engineer corrects the plan before code is written, and the approved version is committed as `plan.md` for later stages to check against |

---

## Getting Started

- **Prerequisites:** The intent artifact (`intent.md` or `spec.md`) if one exists.
  `AGENT.md` helps the agent understand the codebase conventions.
- **Infrastructure:** An AI coding agent with read access to the repository.

---

## How to Execute It

1. Start the agent session in **plan mode** (read-only; no file edits allowed).
2. Give the agent `intent.md` and `spec.md` and ask for an implementation plan
   that names: the files that change, the order of work, and the tests that prove
   it works.
3. Interrogate the plan:
   - "What could this change break?"
   - "Which step is most risky?"
   - "What other approaches did you consider and reject?"
4. Iterate until an engineer who has never seen the conversation could implement
   the change from `plan.md` alone.
5. Commit the approved plan as `plan.md`. The plan joins the audit trail. The
   PR review stage checks the eventual diff against it.
6. Accept the plan and let the agent implement. With a solid plan, implementation
   is often a single pass.
7. When implementation departs from the plan, update `plan.md` in the same
   commit. Consider a hook that enforces synchronization between `plan.md` and
   the diff.

---

## `plan.md` Template

```markdown
# Plan: [Title] (from intent.md [date])

## Files that change
List each file and whether it is new, modified, or deleted.

## Order of work
Numbered steps. Each step is independently verifiable.

## Risks
List what the change could break and how the risk is mitigated.

## Proof
The tests that prove the change works. Name the test file and the cases.
Reference the acceptance criteria from spec.md.
```

### Example

```markdown
# Plan: claims status self-service (from intent.md 2026-06-02)

## Files that change
- portal/src/claims/StatusPanel.tsx  (new)
- claims-api/routes/status.py        (modified)
- claims-api/tests/test_status.py    (new)

## Order of work
1. Add the status endpoint behind existing auth.
2. Build the panel component against the endpoint.
3. Wire the panel into the portal nav.

## Risks
The claims-core API rate-limits at 50 rps; the panel must cache.

## Proof
test_status.py covers the four claim states.
Screenshot matches the approved mock from spec.md.
```

---

## Auto Mode

Once the guardrails from later plays are mature — a tuned `AGENT.md`, skills
that encode policy, hooks that block unsafe actions, and a test suite the agent
can run — plan mode can shift into **auto mode** for routine work:

- Engineer iterates on and approves the plan.
- Agent then applies each change without a per-edit confirmation prompt.
- The review shifts from watching edits happen to reviewing committed artifacts.

Auto mode enables parallelism across worktrees and is fundamental to running
the SDLC autonomously (Stage 6: Maintain).

---

## Governance Considerations

- Design review happens before any code is generated, when changing course is
  still editing a document, not reverting commits.
- Plan mode enforces this structurally: the agent cannot edit files until the
  engineer accepts the plan.
- The plan and its revisions are logged with the identity of who accepted it.
- Routine changes are approved by the engineer. Higher-risk changes go to a
  tech lead or architect.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Share of changes that merge from first implementation pass; time from plan approval to merged PR | PR metadata |
| **Lagging** | Rework cycles per change; how often the merged diff still matches `plan.md` | PR metadata + `git diff plan.md` |
