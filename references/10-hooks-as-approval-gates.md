# Stage 5 · Deploy: Hooks as Approval Gates

## What This Play Does

Hooks are scripts that run before or after an agent action. They can:
- **Allow** the action silently (no human involved)
- **Block** the action with an explanation
- **Ask** — pause the action until a specific person approves

This play formalizes the organization's human approval gates as hooks, so every
required sign-off happens automatically, every time, for every session — without
relying on engineers to remember to ask.

Hooks also appear in earlier stages: blocking edits to migrations and
infrastructure files without a change ticket (Stage 3: Build), and preventing
the agent from editing test files during a fix task (Stage 4: Test).

---

## What Changes

| Traditional | AI-native |
|---|---|
| Governance occurs in review cycles, applied inconsistently — dependent on the reviewer remembering to check | Governance is enforced as the agent acts. The hook runs for everyone, every time, with no variance |

---

## Getting Started

- **Prerequisites:** None. This is a top-row play.
- **Infrastructure:** A written list of the approvals your change process
  requires. Your agent's hook mechanism (most production-grade agents support
  pre-tool and post-tool hooks via a settings file).

---

## How to Execute It

1. Engineering leadership, with change management and compliance, lists the
   human approval gates that must survive: release authorization, change
   management sign-off, edits to protected paths, production deploys.
2. A platform engineer expresses each gate as a hook script that:
   - Reads the pending action from stdin or environment variables
   - Exits `0` to allow, exits `2` (or non-zero) to block, or prompts a human
     and waits for their response before returning
3. Team-level hooks go in the project's agent settings file (e.g.,
   `.agent/settings.json` or `.claude/settings.json`), checked into Git.
4. Non-negotiable, compliance-required hooks go in **managed settings** — a
   configuration file deployed by the platform or IT team that engineers
   cannot edit or override.
5. Every blocking hook must explain itself: the reason and the route to approval
   must appear in the agent's output so the engineer knows what to do.

---

## Hook Script Templates

### Production deploy gate

```bash
#!/bin/bash
# Blocks production deploys unless a release authorization token is set.
# Place at: .agent/hooks/production-gate.sh

cmd=$(cat /dev/stdin | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('command',''))" 2>/dev/null || echo "")

if echo "$cmd" | grep -q "deploy" && echo "$cmd" | grep -q "production"; then
  if [ -z "$RELEASE_APPROVAL" ]; then
    echo "Production deploys require a release authorization." >&2
    echo "Set RELEASE_APPROVAL=<change-ticket-id> and retry." >&2
    exit 2   # exit 2 blocks the action; message goes to the agent
  fi
fi
exit 0
```

### Protected file gate

```bash
#!/bin/bash
# Blocks edits to migration files unless a change ticket is set.
# Place at: .agent/hooks/migration-gate.sh

file=$(cat /dev/stdin | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('file_path',''))" 2>/dev/null || echo "")

if echo "$file" | grep -qE "migrations/|schema/|infra/"; then
  if [ -z "$CHANGE_TICKET" ]; then
    echo "Edits to migrations, schema, and infra require a change ticket." >&2
    echo "Set CHANGE_TICKET=<ticket-id> and retry." >&2
    exit 2
  fi
fi
exit 0
```

### Test file protection (during fix tasks)

```bash
#!/bin/bash
# Blocks edits to test files during a fix branch.
# Place at: .agent/hooks/test-protection.sh

file=$(cat /dev/stdin | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('tool_input',{}).get('file_path',''))" 2>/dev/null || echo "")
branch=$(git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "")

if echo "$file" | grep -qE "(test_|_test\.|\.spec\.|tests/)"; then
  if echo "$branch" | grep -q "^fix/"; then
    echo "Test files are protected on fix/ branches." >&2
    echo "Fix the code, not the test. Raise a separate PR to update tests." >&2
    exit 2
  fi
fi
exit 0
```

---

## Hook Registration

### Team-level (in Git, engineers can adjust)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "${PROJECT_DIR}/.agent/hooks/production-gate.sh" }
        ]
      }
    ]
  }
}
```

### Managed settings (platform-deployed, engineers cannot override)

For regulated environments, the platform team deploys hooks via the admin
console or MDM. Key managed settings to consider:

```json
{
  "permissions": {
    "deny": [
      "Read(.env*)", "Read(./secrets/**)",
      "WebFetch", "Bash(curl *)", "Bash(wget *)"
    ],
    "allow": [
      "Bash(git *)", "Bash(make build)",
      "Bash(make test)", "Bash(make lint)"
    ],
    "disableBypassPermissionsMode": "disable"
  },
  "allowManagedPermissionRulesOnly": true,
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false,
    "network": {
      "allowedDomains": ["git.internal.example.com", "registry.npmjs.org"]
    },
    "credentials": {
      "files": [
        { "path": "~/.ssh", "mode": "deny" },
        { "path": "~/.aws/credentials", "mode": "deny" }
      ]
    }
  },
  "allowManagedHooksOnly": true
}
```

**What each setting controls:**

| Setting | Control objective |
|---|---|
| `permissions.deny` | Keeps secrets out of agent context; blocks arbitrary network egress via tools |
| `permissions.allow` | Pre-approves the safe inner loop so deny list does not create prompt fatigue |
| `disableBypassPermissionsMode` + `allowManagedPermissionRulesOnly` | No engineer, project file, or CLI flag can widen the rules |
| `sandbox.enabled` + `failIfUnavailable` | OS-level isolation; agent refuses to start if sandbox cannot initialize |
| `sandbox.network.allowedDomains` | Blocks network egress at OS level — catches what tool-level denies miss |
| `sandbox.credentials` | Prevents sandboxed shell from reading SSH keys or cloud credentials |
| `allowManagedHooksOnly` | Only hooks in managed settings run; project and local hooks are blocked |

---

## Governance Considerations

- The gate condition is enforced every time, for everyone — no variance by
  engineer or time of day.
- Allow and block decisions are logged with a timestamp.
- The hook itself defines what counts as approval (change ticket ID, release
  manager token, etc.) — ambiguity is eliminated.
- Non-negotiable hooks in managed settings cannot be switched off by an engineer.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Time spent waiting on each approval gate; allow/block counts per gate | Agent observability export (OpenTelemetry) |
| **Lagging** | Gate violations reaching production before and after hooks were introduced | Incident tracker |
