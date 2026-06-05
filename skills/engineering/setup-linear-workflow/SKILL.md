---
name: setup-linear-workflow
description: Use when configuring a repository-owned WORKFLOW.md contract for a Linear-backed unattended agent runner.
disable-model-invocation: true
---

# Setup Linear Workflow

Configure the repository `WORKFLOW.md` contract used by an unattended runner that dispatches Linear issues to coding-agent workspaces.

## Boundary

This skill configures the repository workflow contract only. It does not implement or verify the Symphony runtime. Runtime conformance is owned by the runner program described in `references/symphony-service-spec.md`.

Use the reference spec as the source of truth for the `WORKFLOW.md` contract, especially file format, front matter fields, prompt variables, and runtime-policy documentation. Do not copy the full reference spec into `WORKFLOW.md`; the workflow file should contain only repo runtime configuration and the per-issue prompt template.

## Process

### 1. Explore

Read the repo's existing configuration before asking questions:

- `references/symphony-service-spec.md` — use Sections 5 and 6 for the workflow contract, and Section 18 as a reminder of which guarantees belong to the runtime rather than this skill.
- `docs/agents/issue-tracker.md`, if present — confirm the project issue tracker is Linear and preserve its issue-publishing conventions.
- `docs/agents/triage-labels.md` — understand agent-facing role and label conventions.
- `WORKFLOW.md` at the repo root — parse YAML front matter separately from the prompt body.
- `AGENTS.md` and `CLAUDE.md` — look for existing agent workflow instructions.

If the configured issue tracker is not Linear, stop and report that no Linear workflow contract was written.

If `WORKFLOW.md` exists, preserve its prompt body and valid configured values unless the user explicitly asks to change them. Re-validate existing runtime fields before preserving them:

- `tracker.project_slug` must be a Linear project slugId, not a URL, UUID, team key, or display name.
- `tracker.api_key` must be an environment-variable reference such as `$LINEAR_API_KEY`.
- `tracker.active_states` and `tracker.terminal_states`, if present, must be intentional project states rather than copied examples.

Only read runtime configuration from YAML front matter; do not infer runtime config from the Markdown prompt body.

### 2. Confirm Runtime Contract

Ask only for values that are missing or ambiguous. Confirm:

- Linear project slugId for `tracker.project_slug`; do not use a URL, UUID, team key, or display name as a substitute.
- Linear API key source for `tracker.api_key`; write an environment-variable reference such as `$LINEAR_API_KEY`. Do not write raw tokens.
- Active states for `tracker.active_states`. These are the Linear states the runner may dispatch.
- Terminal states for `tracker.terminal_states`. These states stop work and allow terminal workspace cleanup.
- Handoff states, if any. These are workflow-defined review or handoff states; they are not the same as terminal states unless the user says so.
- Polling interval, if different from the runtime default.
- Workspace root and workspace preparation hooks, if this repo needs checkout/sync/population behavior.
- Agent concurrency, max turns, and retry backoff, if different from runtime defaults.
- Codex command, approval policy, sandbox policy, turn timeout, read timeout, and stall timeout, if the repo needs explicit values.
- GitHub handoff requirements, if the workflow expects agents to push branches or create PRs. Record that the runner must provide `GH_TOKEN` or `GITHUB_TOKEN` through the process environment and that Codex turns must allow network access; do not write raw tokens.
- User-input handling policy for unattended runs. Record this in the prompt body unless the project has a documented workflow extension field for it.

Do not ask for PRD/AFK/HITL labels unless the repo already uses that taxonomy in `docs/agents/triage-labels.md`, existing Linear labels, or user-provided instructions. Those labels are project conventions for prompts and ticket writing; they are not core dispatch gates unless a documented workflow extension implements label filtering.

### 3. Write WORKFLOW.md

Create or update the repository root `WORKFLOW.md`. The file is Markdown with optional YAML front matter. Put runtime configuration in YAML front matter and the per-issue agent prompt template in the Markdown body.

Required Linear dispatch front matter:

```yaml
---
tracker:
  kind: linear
  endpoint: "https://api.linear.app/graphql"
  api_key: "$LINEAR_API_KEY"
  project_slug: "<linear-project-slugId>"
---
```

Only write values that are known. If a required dispatch value is unknown, leave the file unmodified and report the missing field.

`tracker.active_states` and `tracker.terminal_states` may be omitted to use runtime defaults. If the user confirms concrete project states, add them explicitly:

```yaml
tracker:
  active_states:
    - <active-state>
  terminal_states:
    - <terminal-state>
```

Do not copy example state names into `WORKFLOW.md` as project facts.

Prompt body requirements:

- The body is the per-issue prompt template.
- It may reference the normalized `issue` object and `attempt` value supplied by the runner.
- It should describe project-specific ticket handling, validation, handoff, and tracker-write behavior.
- It should not restate the runner implementation spec. Keep implementation details in the reference file and only include instructions the agent needs while working an issue.
- If PRD/AFK/HITL conventions are used, define them in the prompt body or an explicitly documented extension section; do not imply that labels alone gate dispatch.
- If unattended runs must not wait for human input, define the user-input handling policy in the prompt body or in a documented extension field.

Optional front matter fields may be recorded when known:

```yaml
polling:
  interval_ms: 30000
workspace:
  root: "<workspace-root>"
hooks:
  after_create: |
    <optional shell script>
  before_run: |
    <optional shell script>
  after_run: |
    <optional shell script>
  before_remove: |
    <optional shell script>
  timeout_ms: 60000
agent:
  max_concurrent_agents: 10
  max_turns: 20
  max_retry_backoff_ms: 300000
  max_concurrent_agents_by_state: {}
codex:
  command: "codex --config shell_environment_policy.inherit=all app-server"
  approval_policy: "<implementation-defined>"
  thread_sandbox: "<implementation-defined>"
  turn_sandbox_policy:
    type: workspaceWrite
    networkAccess: true
  turn_timeout_ms: 3600000
  read_timeout_ms: 5000
  stall_timeout_ms: 300000
```

Do not write placeholder optional values into `WORKFLOW.md` as if they were real configuration. Use the optional block as a prompt for collecting concrete values. Codex environment inheritance and `networkAccess: true` are appropriate when the workflow requires CLI tools such as `gh` to read process-provided credentials and reach GitHub; omit them for workflows that intentionally run without inherited secrets or external network.

### 4. Update References

If needed, add project-specific tracker expressions to `docs/agents/triage-labels.md`, but do not replace the canonical triage roles. Workflow-specific labels are additional project conventions, not a new core dispatch model.

Do not create `docs/agents/workflow.md`, and do not add pointers to it. `WORKFLOW.md` is the single source of truth for the unattended runner contract. `docs/agents/issue-tracker.md`, when present, is for Matt Pocock engineering skills and issue-publishing conventions, not Symphony runtime configuration.

### 5. Validate Contract Output

Before reporting done, check the generated contract against the reference spec:

- `WORKFLOW.md` exists at the repo root unless the user provided an explicit runtime path.
- YAML front matter, if present, parses as a map.
- Required Linear dispatch fields are concrete: `tracker.kind`, `tracker.api_key`, and `tracker.project_slug`.
- `tracker.api_key` is an environment-variable reference, not a raw token.
- No placeholder optional values were written as real config.
- Runtime config lives in front matter, not the Markdown prompt body.
- Prompt body is present, or the report explicitly says the runner will need its default prompt behavior.
- Approval, sandbox, and user-input policy are documented when the repo needs unattended behavior.
- Workflows that require GitHub PR handoff document both prerequisites: Codex can access the network, and the runner process provides a valid `GH_TOKEN` or `GITHUB_TOKEN` without storing the token in `WORKFLOW.md`.

Do not report runtime conformance. Report the workflow contract status and any runtime assumptions separately.

### 6. Done

Report the files changed and the exact runtime contract recorded. If `WORKFLOW.md` was not written because required values were missing, report those missing values. Include `runtime conformance: not verified by this skill` unless a separate runner verification was actually performed.
