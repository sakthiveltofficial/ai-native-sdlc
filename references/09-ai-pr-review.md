# Stage 5 · Deploy: AI in the PR Review Loop

## What This Play Does

Adds the AI agent as the **first reviewer** on every pull request. The agent
reads the diff, checks it against `plan.md`, applies the active skills (security,
architecture, style), and posts structured findings before any human opens the
PR. Human reviewers concentrate on intent, risk, and judgment calls — the agent
handles the mechanical checks.

Separation of duties is preserved: the agent that wrote the code is a different
instance from the agent that reviews it. Neither instance can approve the PR —
only a human with the right permissions can do that.

---

## What Changes

| Traditional | AI-native |
|---|---|
| Humans review every line of code. Governance occurs in review cycles, often inconsistently. A reviewer may miss a pattern on a Friday afternoon that they would catch on Monday morning | Layers of agentic review; human review reserved for regulated and critical code. The agent applies the same policies every time, at the same depth, with no variance for time of day or reviewer fatigue |

---

## Getting Started

- **Prerequisites:** Continuous evals (Stage 4). Parallel sessions / subagents
  (Stage 3) help the review agent run independently. Skills help significantly —
  the agent applies them as review criteria.
- **Infrastructure:**
  - A CI/CD system that can run a script on PR open/update
  - A way to post comments to PRs via the platform API (GitHub, GitLab, etc.)
  - A way to invoke the AI agent non-interactively

---

## How to Execute It

1. Create a `REVIEW.md` file at the repo root. This is the agent's review
   policy — the instructions it follows when reviewing any PR in this repo.
2. Configure CI to run the review agent automatically on every new PR and on
   every push to an open PR.
3. The agent reads `REVIEW.md`, the diff, and any active skills, then posts
   its findings as a PR comment in the standard format (see below).
4. The agent rates the PR against the criteria in `REVIEW.md`. If it rates
   the PR as ready for human review, the PR moves forward. If not, the author
   addresses the findings first.
5. A human reviewer with merge rights reviews the agent's findings alongside
   the diff and makes the final approval decision. The agent's output informs
   but does not replace human judgment.
6. Any finding the human reviewer disagrees with is fed back into `REVIEW.md`
   or the active skills — the policy improves with each review cycle.

---

## `REVIEW.md` Template

```markdown
# PR Review Policy

## Scope
Review all changes to [scope: the full repo / src/ only / excluding generated files].

## Findings format
Use this structure for every finding:

**[IMPORTANT | Nit]** `path/to/file.ext` line N
> Quote the relevant line(s).
One sentence explaining the issue and the policy or principle it violates.
If IMPORTANT: one sentence suggesting the smallest fix.

## What IMPORTANT means
Reserve IMPORTANT for findings that would break behavior, leak data,
or breach a stated policy. Style and naming are nits.

## Cap nits
Report at most five nits per review. Summarize the rest as a count.

## Do not report
- Files under [generated file path]
- Anything already enforced by the linter or formatter in CI

## Check against plan.md
If a plan.md exists for this change, verify the diff matches it.
Flag any departure from the plan as IMPORTANT with the specific discrepancy.

## Skills to apply
List the skill names the review must apply, e.g.:
- security-policy
- architecture-hexagonal
- api-versioning
```

---

## PR Review Comment Format

The agent posts a structured comment so findings are easy to triage:

```markdown
## Agent Review

**Plan alignment:** ✅ Diff matches plan.md  (or ❌ Departs from plan — see findings)

### IMPORTANT findings (must address before merge)
1. `claims-api/routes/status.py` line 42  
   > password = "hardcoded_value"  
   Hardcoded credential. Reference via environment variable per SEC-002.

### Nits (optional, max 5 shown)
1. `portal/src/claims/StatusPanel.tsx` line 18 — prefer `const` over `let` (value never reassigned).

_3 additional nits suppressed._

**Verdict:** ⚠️ Address IMPORTANT findings before human review.
```

---

## Governance Considerations

- **Separation of duties**: the agent that authored the code has no way to
  approve the PR. Branch protection rules enforce human approval.
- **Policy applied consistently**: `REVIEW.md` is the same for all PRs; there
  is no variance across reviewers or time of day.
- **Audit record**: findings, fixes, ratings, and human approvals are all in
  the PR history — the PR is the audit record.
- **REVIEW.md is version controlled**: changes to the review policy are reviewed
  in PR like any other code change.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Time to first review comment (should drop to minutes); share of review comments resolved without human touching the branch | PR metadata |
| **Lagging** | Defects and vulnerabilities caught before merge vs. those escaping to production | PR history + incident tracker |
