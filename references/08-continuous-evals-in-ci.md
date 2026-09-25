# Stage 4 · Test: Continuous Evals in CI

## What This Play Does

Adds a persistent eval suite to CI that runs automatically on every change.
Each eval is a question the agent answers about the codebase. The suite grows
by one eval per bug that reaches production — so the history of incidents becomes
a regression guard that runs on every pull request.

Unlike unit tests (which check that code does what the developer intended), evals
check that the AI agent's outputs meet expectations on realistic tasks: "Does the
agent correctly handle this edge case?" or "Does the refactored module still pass
the security policy check?"

---

## What Changes

| Traditional | AI-native |
|---|---|
| QA gates at stage boundaries — a human tester runs a checklist before release | Continuous evals woven through implementation — the agent's outputs are checked automatically on every PR, in the same pipeline that checks the code |

---

## Getting Started

- **Prerequisites:** `AGENT.md` (so the agent under test has proper context).
  A feedback loop (Stage 4 – Feedback Loop) so the agent can run evals locally.
- **Infrastructure:**
  - A CI system (GitHub Actions, GitLab CI, Jenkins, etc.)
  - A way to invoke the AI agent non-interactively (most agents support a
    `--print` or headless mode; alternatively use the provider's API or SDK)
  - A place to store eval definitions (a directory in the repo, e.g., `evals/`)

---

## How to Execute It

1. **Start with one eval per known bug.** After each incident or production bug,
   write an eval that would have caught it. The eval must be runnable in CI
   without human input.
2. **Write evals as structured files** with an input, an expected output or
   behavior description, and a grading method (exact match, model-graded, or
   assertion-based).
3. **Run evals in CI** on every pull request that touches agent-facing code
   (prompts, skills, `AGENT.md`, evaluation criteria). Gate the PR on eval pass.
4. **Use model-graded evals sparingly** for outputs that cannot be exactly
   matched. For graded evals, the grading prompt must itself be versioned and
   reviewed.
5. **Track eval results over time.** A regression (eval that used to pass now
   fails) is a signal that a change broke something the eval was guarding.

---

## Eval File Format

```markdown
# Eval: [Short descriptive name]

## Input
[The task or prompt given to the agent under test.]

## Context
[Any files, code snippets, or system state the agent should see.]

## Expected behavior
[What the agent must do or produce. Be specific and testable.]

## Grading method
exact_match | assertion | model_graded

## Grading details
[For assertion: the assertion script or regex.]
[For model_graded: the grading prompt and pass/fail threshold.]

## Source
[Bug ID, incident, or requirement this eval guards against.]
```

---

## Example Evals

### Assertion-based eval

```markdown
# Eval: no-hardcoded-db-password

## Input
Add a new database connection for the reporting service to config/db.py.

## Context
Existing: config/db.py (provided in context)
Requirements: reporting service uses host=db-reporting, db=reports

## Expected behavior
The agent must not write a password literal into any source file.
It must reference the password via an environment variable.

## Grading method
assertion

## Grading details
assert "password" not in diff or "os.environ" in diff or "getenv" in diff

## Source
INC-0042: Credentials committed to repo by junior developer
```

### Model-graded eval

```markdown
# Eval: spec-md-covers-constraints

## Input
Read the attached intent.md and produce a spec.md.

## Context
intent.md: [provided]
Active skills: security-policy-v2, brand-guidelines-v1

## Expected behavior
The spec.md must explicitly list the constraints imposed by each active skill.
It must flag any areas where the intent conflicts with the skills.

## Grading method
model_graded

## Grading details
Grading prompt: "Does the spec.md explicitly reference the security-policy-v2
and brand-guidelines-v1 skills? Does it flag any conflicts? Score 1 (pass) if
both are true, 0 (fail) otherwise."
Pass threshold: 1

## Source
Requirement: REQ-017 — policy conflicts must be surfaced in the spec
```

---

## CI Configuration Sketch

```yaml
# .github/workflows/evals.yml (GitHub Actions example — adapt for your CI)
name: Agent Evals
on: [pull_request]
jobs:
  evals:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run eval suite
        run: |
          # Replace with your agent's headless invocation
          python scripts/run_evals.py --eval-dir evals/ --agent-cmd "agent --headless"
        env:
          AGENT_API_KEY: ${{ secrets.AGENT_API_KEY }}
```

---

## Governance Considerations

- Evals are version controlled — changes to the eval suite are reviewed in PR
  like any other code change. An eval can only be removed with an explicit PR
  showing why it is no longer needed.
- The CI run that executes evals is deterministic and logged. Results are
  attached to the PR and visible to reviewers and auditors.
- Model-graded evals introduce a second AI judgment; the grading prompt is
  itself versioned and subject to the same review gate.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Eval coverage: share of past production incidents with an active eval | Incident tracker cross-referenced with eval files |
| **Lagging** | Production incidents in classes already covered by evals — should trend to zero | Incident tracker |
