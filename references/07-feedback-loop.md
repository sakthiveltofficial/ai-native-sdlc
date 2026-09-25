# Stage 4 · Test: Give the Agent a Feedback Loop

## What This Play Does

Every agent session should have a way to verify its own work — tests, a build
check, a screenshot diff, a linter. The session checks its own output and fixes
its own mistakes before a human sees them. The signal that code works arrives
at the agent, not days later at CI or weeks later in production.

> **Note:** The feedback loop runs throughout the whole task. A verifier subagent
> (Stage 3) provides one additional check in a fresh context window after the
> session believes it is done. Both are complementary — use both.

---

## What Changes

| Traditional | AI-native |
|---|---|
| Signal that code works arrives late: CI minutes later, a tester days later, production weeks later. With an agent producing code, a late signal means a person must check all of its output — that person becomes the bottleneck | The session is given a way to check its own work before a person sees it. The agent runs tests, build, and lint, then iterates until they pass. What reaches the engineer has already passed the check |

---

## Getting Started

- **Prerequisites:** None. This is a top-row play.
- **Infrastructure:**
  - A test suite and a build that run locally with a single command each.
  - For UI work: a browser tool or screenshot utility the agent can call.

---

## How to Execute It

1. **Wrap verification in one command.** If checking the work today requires a
   sequence of commands and environment knowledge, wrap it in a single target
   (`make verify`, `npm test`, `./scripts/check.sh`) that exits non-zero on
   failure. One command means the agent never has to guess what to run.

2. **Document the commands in `AGENT.md`.** In the Commands section, list each
   check command with an example of healthy output. The agent will run these
   before reporting any task complete.

3. **State a quantifiable target.** Tell the agent what success looks like so it
   can check without asking you:
   - "All tests in `test_claims.py` pass."
   - "The endpoint returns 200 with the `status` field present."
   - "The screenshot matches the attached mock within a 5 px tolerance."

4. **For bug fixes, write the failing test first.** Ask the agent to reproduce
   the bug as a test, run it, confirm it fails for the right reason, then commit
   that test. Only then ask the agent to make it pass without editing the test.
   A test that existed before the fix, and that the agent could not rewrite, is
   proof the bug is gone.

5. **For UI work, close the loop visually.** Give the agent a browser or
   screenshot tool and the design mock. Let it iterate: implement, screenshot,
   compare, adjust. Two or three rounds is normal.

6. **Make verification part of "done."** Add to `AGENT.md`:
   ```
   Run all verification commands before reporting any task complete, and
   paste the output. If a test fails, fix the code, not the test.
   ```

7. **Protect the feedback loop itself.** An agent fixing code must not be able
   to weaken its own verification. Use a hook that blocks edits to test files
   during a fix task, or enforce this in PR review by rejecting any change that
   touches a test file on a fix branch.

---

## `AGENT.md` Verification Block

```markdown
## Verifying your work

- Build: <build command>   (must finish with "<success message>")
- Test:  <test command>    (all green; never skip or delete a failing test)
- Lint:  <lint command>    (zero warnings)

Run all three before reporting any task complete, and paste the output.
If a test fails, fix the code, not the test.
```

---

## Governance Considerations

| Control | How it is enforced |
|---|---|
| Verification before task completion | Instruction in `AGENT.md`; optionally a hook that blocks the session reporting done without test output |
| Agent cannot weaken its own tests | Hook that blocks edits to test files during a fix task |
| Evidence of verification | Literal output of `make test` / `npm test` pasted in the session transcript, forwarded to the observability stack |
| Who approves | The code owner reviewing the PR; they can focus on intent and risk because mechanical evidence is already attached |

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | First-pass CI success rate for agent-written changes | CI system |
| **Lagging** | Review time per PR (should fall once tests catch what reviewers used to catch); change failure rate | PR metadata; incident tracker |
