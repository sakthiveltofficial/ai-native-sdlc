# Stage 5 · Deploy: CI/CD Integration and Deployment

## What This Play Does

Connects the agentic review loop to the CI/CD pipeline so that agent-verified
changes flow to production without additional manual handoffs. The agent's
verdict in PR review becomes a formal pipeline gate. Deployment itself can be
triggered by a merged PR, with the agent monitoring the rollout and flagging
anomalies.

This play requires PR review (Stage 5) and hooks (Stage 5) to be in place first.

---

## What Changes

| Traditional | AI-native |
|---|---|
| Humans review every line, then manually trigger deploys. Release managers coordinate across teams on a schedule | Agent review is a formal gate in the pipeline. Merges trigger automated deployment. The agent monitors the rollout; anomalies feed back as `intent.md` (Stage 6) |

---

## Getting Started

- **Prerequisites:** AI in the PR review loop (Stage 5). Hooks as approval
  gates (Stage 5).
- **Infrastructure:**
  - An existing CI/CD pipeline (GitHub Actions, GitLab CI, Jenkins, Tekton, etc.)
  - A way to invoke the AI agent non-interactively as a pipeline step
  - Deployment tooling with an API or CLI the pipeline can call

---

## How to Execute It

### Step 1 — Add agent review as a required CI check

Configure the CI system to run the agent reviewer on every PR. Make the check
required: the PR cannot merge until the agent posts its verdict and no IMPORTANT
findings remain unresolved.

```yaml
# Example: GitHub Actions
name: Agent PR Review
on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  agent-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0          # full history so the agent can read context
      - name: Run agent review
        run: |
          git diff origin/${{ github.base_ref }}...HEAD > /tmp/diff.patch
          agent --headless --print \
            --context REVIEW.md \
            --context /tmp/diff.patch \
            "Review this diff per REVIEW.md. Output findings in the standard format.
             Exit 1 if there are any IMPORTANT findings." \
          | tee /tmp/review.md
        env:
          AGENT_API_KEY: ${{ secrets.AGENT_API_KEY }}
      - name: Post review comment
        uses: peter-evans/create-or-update-comment@v4
        with:
          issue-number: ${{ github.event.pull_request.number }}
          body-file: /tmp/review.md
```

### Step 2 — Gate merge on agent verdict

In your repository settings, add the `agent-review` check to the required
status checks for the main/production branch. The PR cannot merge if the check
fails (IMPORTANT findings present).

### Step 3 — Automate deployment on merge

Configure the pipeline to deploy automatically when a PR merges to the main
branch, after the full test suite passes:

```yaml
name: Deploy on merge
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run full test suite
        run: make test
      - name: Run eval suite
        run: python scripts/run_evals.py --eval-dir evals/
      - name: Deploy
        run: make deploy-staging    # or deploy-production per your flow
        env:
          RELEASE_APPROVAL: ${{ secrets.RELEASE_APPROVAL_TOKEN }}
```

### Step 4 — Agent-monitored rollout

After deploying, run a monitoring agent step that checks the key metrics for
the change (error rate, latency, test results). If metrics breach a threshold,
the step fails and the pipeline posts an alert. See Stage 6 (Closing the Loop)
for the full monitoring pattern.

---

## Running the Agent Non-interactively in CI

Most agents support a headless or non-interactive mode. Common patterns:

| Agent | Non-interactive invocation |
|---|---|
| Claude Code | `claude --headless --print "<prompt>"` |
| OpenAI Codex CLI | `codex --full-auto "<prompt>"` |
| Aider | `aider --yes --no-stream --message "<prompt>"` |
| Custom agent via API | Call the provider's API directly from a Python script |

Store API keys as CI secrets, never in source code.

---

## Artifact Traceability in the Pipeline

Every pipeline run should record the artifacts that triggered it:

| Artifact | Where to record |
|---|---|
| `intent.md` commit SHA | PR description; pipeline run metadata |
| `spec.md` commit SHA | PR description |
| `plan.md` commit SHA | PR description |
| Agent review output | PR comment; pipeline artifact store |
| Eval results | Pipeline artifact store; PR status check |
| Deploy commit SHA | Deployment record; monitoring dashboard |

This chain of SHAs is the audit trail: who asked for what, what the agent
produced, and who approved it.

---

## Legacy System Integration

If your organization uses a change management tool (ServiceNow, Jira Service
Management), integrate the pipeline with it:

- On PR merge, create or update the change record via the tool's API or an
  MCP connector.
- Record the deploy commit SHA and agent review summary in the change record.
- On successful deploy, close the change record automatically.

---

## Governance Considerations

- The agent review check is a formal pipeline gate — it runs for every change,
  with no exceptions.
- Human approval (branch protection rule) is still required for production
  merges. The agent informs; a human approves.
- The pipeline run log is the audit record: timestamp, inputs, agent output,
  test results, deploy target, and approver.
- Rollback is also automated: if post-deploy monitoring detects a breach, the
  pipeline triggers the pre-approved rollback runbook.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Deploy frequency; time from merged PR to production deploy | Pipeline metrics |
| **Lagging** | Change failure rate (deploys that required rollback) | Pipeline + incident tracker |
