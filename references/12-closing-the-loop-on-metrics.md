# Stage 6 · Maintain: Closing the Loop on Metrics

## What This Play Does

Connects production monitoring back to Stage 1 so the loop is truly closed.
A detection script watches one or more metrics against a rolling baseline. When
a metric breaches a control band, the agent is invoked automatically: it
diagnoses, writes an `intent.md`, and routes the finding to the triage queue —
all without a person starting the loop.

Human attention concentrates on the triage queue: review, route, dismiss, or
escalate. The loop feeds itself.

---

## What Changes

| Traditional | AI-native |
|---|---|
| Humans watch production for bugs. An incident at 10 pm gets a tired engineer paged at 11 pm. Post-mortems are written days later and rarely become systematic prevention | Agents monitor live deployments. Any breached control band is diagnosed and written back into the loop as a new `intent.md`. Humans triage the queue, not the raw alerts |

---

## Getting Started

- **Prerequisites:** CI/CD integration (Stage 5). Capture intent (Stage 1) —
  the agent writes findings as `intent.md`.
- **Infrastructure:**
  - A metrics store the detection script can query (Prometheus, Datadog, the
    CI system's API, CloudWatch, or equivalent)
  - Read access to the repository
  - A way to run the agent non-interactively on a schedule or webhook trigger
  - (Optional) An MCP server connecting the agent to the monitoring stack and
    the project management tool

---

## How to Execute It

1. **Pick one metric** with a stable rolling baseline (CI test failure rate,
   post-deploy 5xx rate, PR cycle time, p99 latency). Start with one.

2. **Write the detection script.** Calculate mean and standard deviation over
   a rolling window (e.g., 30 days). Apply statistical process control rules
   (Western Electric or similar) so the bands catch slow drift as well as spikes.
   The detection logic is entirely deterministic — no model involved at this step.

3. **Define response tiers** in version-controlled config (`bands.yaml`):
   - At 1σ: log only.
   - At 2σ: invoke the agent read-only to diagnose; no writes.
   - At 3σ: agent may propose a fix by opening a PR or triggering a pre-approved
     runbook — always through a review gate.

4. **Configure the trigger layer.** Options:
   - A scheduled CI workflow (cron job in GitHub Actions, GitLab CI, etc.)
   - A webhook from the existing monitoring stack (PagerDuty, Alertmanager, etc.)
   - A cron job inside the network calling the agent API

5. **Agent writes its diagnosis as `intent.md`** in the Stage 1 format (problem,
   desired outcome, affected systems, open questions, evidence). The finding
   enters the same triage queue as human-authored intent files.

6. **Service owner or on-call engineer triages the queue.** Options: fix now,
   schedule, or dismiss. Dismissals tune the bands to reduce noise.

7. **When a fix ships, add an eval** for the incident (Stage 4: Evals) so the
   class of issue is guarded against going forward.

---

## `bands.yaml` Example

```yaml
# bands.yaml — monitoring config, version controlled in .agent/monitoring/
metric: ci_test_failure_rate
baseline: rolling_30d
rules: western_electric      # or: simple_sigma, ewma
tiers:
  1sigma:
    action: log
  2sigma:
    action: diagnose
    tools: "Read, Grep, Bash(gh run view *)"    # read-only tools only
  3sigma:
    action: propose
    routes:
      - pull_request          # opens a PR into the normal review gate
      - runbook:rollback-deploy  # triggers a pre-approved runbook
```

---

## `intent.md` Written by the Agent (Production Breach)

```markdown
# Intent: Elevated CI test failure rate — automated detection

## Problem
The CI test failure rate has breached the 3σ band (baseline 4.2%,
current 7-day rate 11.8%) as of 2026-09-25T08:14Z. The spike began
approximately 36 hours after the merge of PR #847 (auth-service refactor).

## Desired outcome
CI test failure rate returns to within 1σ of the rolling 30-day baseline
within 48 hours.

## Affected systems
- auth-service (PR #847 identified as likely culprit)
- ci-runner pool (potential flaky test infrastructure issue — secondary hypothesis)

## Evidence
- Band breach timestamp: 2026-09-25T08:14Z
- Rolling baseline (30d): 4.2% ± 1.1%
- Current rate: 11.8%
- Relevant CI runs: [run-id-1, run-id-2, run-id-3]

## Open questions
1. Is the failure rate correlated with a specific test suite or all suites?
   (Owner: platform-team)
2. Are the failing tests deterministically failing or flaky?
   (Owner: on-call engineer)

## Source
Automated detection — bands.yaml, metric: ci_test_failure_rate
Diagnosis run: [session-id]
```

---

## Worked Examples

| Scenario | Detection | Response |
|---|---|---|
| CI test failure rate breaches 3σ | Detection script | Agent quarantines the flaky test or opens a revert PR; review gate decides |
| Post-deploy 5xx rate breaches 3σ with a deploy in the window | Webhook from monitoring stack | Agent triggers rollback runbook; posts to incident channel |
| PR cycle time trips a drift rule | Scheduled cron | Agent writes a report for engineering leadership; no code change |

---

## Incident Channels and Collaboration

When your team uses a messaging platform (Slack, Microsoft Teams) for incidents,
some AI platforms offer a channel integration that makes the agent a first
responder under its own identity in the channel.

In this pattern:
- Each new incident gets an immediate first-response diagnosis.
- Team members can guide and question the agent in the thread.
- The agent verifies the metric is back at baseline and confirms in the thread.
- The post-mortem is written to a version-controlled lessons file that future
  investigations can read.
- Small, well-bounded fixes arrive as PRs through the review gate. Larger
  findings are written as `intent.md` and enter the triage queue.

---

## Governance Considerations

- **Tier boundaries enforced from config.** `bands.yaml` is version controlled;
  changes are reviewed in PR. The detection logic is deterministic — no model
  decides whether a threshold is breached.
- **Permission tiers.** At 1σ and 2σ, the agent has read-only access. At 3σ,
  the agent can only route through pre-approved runbooks or open PRs — no direct
  production access.
- **Full audit log.** Invocations, findings, and triage decisions are logged with
  timestamps. The PR or runbook execution that follows is also logged.
- **Human triage required.** The agent proposes; the service owner decides.
  Runbooks the agent may trigger were approved in advance.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Time from band breach to `intent.md` in the triage queue, compared to old time from incident to post-mortem action | Detection script log (breach timestamp) vs. `intent.md` commit timestamp |
| **Lagging** | Share of findings that become merged fixes; repeat incidents of the same class (should fall as evals accumulate) | Triage queue vs. PR history; incident tracker |
