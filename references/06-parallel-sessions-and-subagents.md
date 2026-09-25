# Stage 3 · Build: Parallel Sessions and Subagents

## What This Play Does

Instead of one engineer working one task sequentially, an engineer runs several
agent sessions simultaneously — each in its own Git worktree on its own task.
Repeated jobs become **subagents**: scoped agent definitions with their own
context window, system prompt, and tool permissions, launched by the main session
without blocking it.

The engineer's role shifts from implementing to orchestrating.

---

## What Changes

| Traditional | AI-native |
|---|---|
| One engineer, one task at a time. Context-switching while waiting is possible but tiring enough that few people do it | One engineer runs several agent sessions at once, each in its own worktree on its own task. Repeated jobs become subagents. The engineer orchestrates and eventually builds and monitors loops |

---

## Getting Started

- **Prerequisites:** `AGENT.md` (all sessions read it). The feedback loop play
  (Stage 4) helps too — less supervision is needed when a session can verify
  its own work.
- **Infrastructure:** A Git repository (isolation comes from worktrees). Agent
  permission settings tuned so sessions are not waiting on approval prompts for
  commands the organization considers safe.

---

## How to Execute It

### Parallel Sessions

1. Use the implementation plan from the plan mode play (Stage 3) to identify
   which tasks touch different files and can run independently. Tasks that share
   files run sequentially in a single session.
2. Give each independent task its own **worktree** — a separate checkout on its
   own branch — so sessions cannot collide on the same files.
   ```
   git worktree add ../feature-auth -b feature-auth
   git worktree add ../fix-rate-limit -b fix-rate-limit
   # Start one agent session per worktree
   ```
3. Start with two or three sessions. The practical ceiling is how many streams
   one person can review properly. Add sessions only while review quality holds.

### Subagents

4. Identify repeated jobs that appear across many tasks: a code simplifier, a
   behavior verifier, a codebase researcher that reports without polluting the
   main context.
5. Define each subagent as a Markdown file in `.agents/agents/` (or the
   equivalent directory your agent uses). Each file needs:
   - A `name`
   - A `description` of when the main agent should delegate to it
   - The `tools` it is allowed to use
   - Its system prompt / instructions
6. Commit the subagent definitions to Git so the whole team shares them.
7. The main agent launches subagents when their description matches the task,
   without requiring an engineer to intervene.

---

## Subagent Definition Template

```markdown
---
name: <short-name>
description: >
  One sentence from the main agent's perspective: "Use this subagent when …"
  The main agent reads this to decide when to delegate.
tools: <comma-separated list of allowed tools, e.g., Bash, Read, Write>
---

[Clear instructions for what the subagent does and what it must report back.]
[Define what "done" means for this subagent.]
[List any constraints on what the subagent must NOT do.]
```

---

## Example Subagent Definitions

### Verifier subagent

```markdown
---
name: verifier
description: >
  Use this subagent when the main session believes it has completed a task.
  The verifier runs the app and checks behavior in a fresh context window,
  so its verdict is not colored by the assumptions that produced the code.
tools: Bash, Read
---

Start the app with the build command from AGENT.md.
Exercise the changed behavior and the two nearest neighboring flows.

Report:
1. What you ran (exact commands).
2. What you observed (output, UI state, network responses).
3. Any behavior that does not match plan.md.

Do not fix anything. Report only.
```

### Researcher subagent

```markdown
---
name: researcher
description: >
  Use this subagent to explore the codebase and answer a specific question
  before the main session acts. Keeps research out of the main context window.
tools: Read, Grep, Bash(git log *), Bash(git blame *)
---

Answer the question given to you by reading the codebase.
Cite the file and line number for every claim you make.
Do not modify any file.
Return a concise summary: answer first, then supporting evidence.
```

### Simplifier subagent

```markdown
---
name: simplifier
description: >
  Use this subagent after the main session's implementation is complete and
  tests pass. Strips needless complexity without changing behavior.
tools: Read, Write, Bash(make test)
---

Read the diff produced by the main session (compare the current branch to main).
Identify any code that is more complex than necessary without being safer or
clearer. Propose and apply simplifications one at a time.
Run the tests after each simplification. Stop immediately if tests fail.
Do not change any test files.
Report what you simplified and why.
```

---

## Governance Considerations

- More sessions mean more output, so controls must come from configuration —
  not from per-session briefings. Hooks and permission settings in the repo
  apply to all sessions automatically.
- What each session does is logged and attributed to the engineer who ran it.
- Subagent tool permissions are narrower than the main session's. Define the
  minimum tool set the subagent needs.
- Subagent definitions in Git are reviewable — policy owners can audit what the
  team's automated helpers are permitted to do.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Concurrent sessions per engineer while review quality holds | Observability export (e.g., OpenTelemetry); session logs |
| **Leading** | Share of engineer time spent steering rather than waiting | Time-tracking or session log analysis |
| **Lagging** | Changes merged per engineer per week, alongside rework rate | PR history |
