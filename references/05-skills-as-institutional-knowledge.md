# Stage 3 · Build: Skills as Institutional Knowledge

## What This Play Does

Skills are Markdown files that encode the team's standards — security policies,
architecture patterns, brand guidelines, testing conventions — in a form that
the agent can read and apply. Instead of hoping each engineer remembers to brief
the agent on the rules, the rules are versioned in Git, distributed to every
session automatically, and updated like code.

Skills are the organization-wide complement to `AGENT.md`. `AGENT.md` covers
what is specific to one repository. Skills cover what applies across all
repositories.

---

## What Changes

| Traditional | AI-native |
|---|---|
| Policies live in Confluence, wikis, or people's heads. Each engineer interprets them differently. Agents have no access to them | Policies are expressed as skill files, versioned in Git, distributed via the agent's plugin or skill system. Every session in every repo applies the same live policy |

---

## Getting Started

- **Prerequisites:** None. This is a top-row play.
- **Infrastructure:** An agent that supports skill files or system prompt
  includes. A shared Git repository (or plugin marketplace) from which skills
  are distributed to engineers.

---

## How to Execute It

1. Identify the policy that causes the most rework or the most review comments.
   Start with one skill.
2. Write the skill as a Markdown file with a YAML front matter block that names
   it and describes when it applies.
3. Publish the skill to a shared location engineers can pull from (a Git repo,
   the agent's plugin marketplace, or a team-managed directory).
4. Set the agent to load the skill automatically for sessions in the relevant
   scope (all repos, specific repos, specific file types).
5. Verify: run the agent on a representative task and confirm it applies the
   policy without being asked.
6. Iterate: add a skill for the next highest-friction policy.

---

## Skill File Format

```markdown
---
name: <short-kebab-case-name>
description: >
  One or two sentences. This is what the agent reads to decide whether
  to apply the skill. Write it from the agent's perspective: "Apply this
  skill when …"
---

# [Skill Title]

[Full instructions the agent follows when this skill is active.]

## Rules
- [Concrete, testable rule]
- [Concrete, testable rule]

## Examples
[Optional: a good example and a bad example, with commentary.]

## References
[Links to the authoritative policy documents this skill encodes.]
```

---

## Example Skills

### Security: No Hardcoded Secrets

```markdown
---
name: no-hardcoded-secrets
description: >
  Apply this skill to every code change. Prevents secrets, credentials,
  API keys, and tokens from being committed to the repository.
---

# No Hardcoded Secrets

## Rules
- Never write a literal secret, API key, password, token, or private key
  into source code or configuration files committed to the repository.
- Reference secrets through environment variables or a secrets manager
  (e.g., AWS Secrets Manager, HashiCorp Vault, a .env file that is
  gitignored).
- If you find an existing hardcoded secret while working on unrelated code,
  flag it as a finding in your output but do not fix it unless explicitly asked.

## What counts as a secret
API keys, passwords, private keys (RSA/EC/etc.), OAuth client secrets,
JWT signing secrets, database connection strings with credentials embedded,
bearer tokens, and webhook secrets.

## References
- OWASP: Sensitive Data Exposure
- Internal policy: SEC-002
```

### Architecture: Hexagonal / Ports-and-Adapters

```markdown
---
name: hexagonal-architecture
description: >
  Apply this skill when adding features or refactoring in repositories
  that follow the hexagonal (ports-and-adapters) architecture pattern.
---

# Hexagonal Architecture Conventions

## Rules
- Domain logic goes in `core/`. It has zero imports from `adapters/` or `api/`.
- Adapters (database, HTTP, message broker) live in `adapters/` and depend on
  interfaces defined in `core/`, never the reverse.
- REST controllers live in `api/` and call `core/` only through use-case classes.
- Do not create cross-layer shortcuts even for "small" changes.

## References
- ADR-004: Why we chose hexagonal architecture
```

---

## Distributing Skills to Engineers

| Method | When to use |
|---|---|
| Shared Git repository | Small teams; engineers clone the skill repo alongside project repos |
| Agent plugin marketplace | Orgs using an agent platform with a marketplace (e.g., Claude Code admin console, Cursor teams) |
| Managed settings injection | Regulated orgs where platform team controls the full agent config via MDM or admin console |

---

## Governance Considerations

- Skills are version controlled. The instructions the agent works under are
  reviewable and auditable at any commit.
- Changes to shared skills are reviewed in PR like code changes — policy changes
  go through the same gate as implementation changes.
- Platform teams can mark certain skills as mandatory via managed settings,
  preventing engineers from disabling them.
- When the Requirements & Design play runs, the agent applies the active skill
  versions and records which versions were used — a complete policy audit trail.

---

## How to Measure It

| Indicator | What to track | Where to find it |
|---|---|---|
| **Leading** | Review comments citing a policy that a skill should have caught, before and after the skill is published | PR comment history |
| **Lagging** | Policy violations reaching production | Incident tracker cross-referenced with skill publish dates |
