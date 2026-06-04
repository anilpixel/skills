# Issue tracker: Linear

Issue tracker operations for this repo use Linear. Prefer the available Linear skill, Linear MCP, or `linear_graphql` tool for Linear operations.

## Project Configuration

Record the actual values for this repo:

```yaml
tracker:
  kind: linear
  endpoint: "https://api.linear.app/graphql"
  api_key: "$LINEAR_API_KEY"
  project_slug: "<linear-project-slugId>"
```

`tracker.project_slug` is the Linear project `slugId`; do not use a Linear UUID, URL, team key, or display name as a substitute.

When generating `docs/agents/issue-tracker.md`, replace placeholders with concrete non-secret values. If the Linear project slugId is unknown, do not write a placeholder as if it were configured; report the missing field instead.

Do not write raw API tokens into this file. Use `$LINEAR_API_KEY` or another documented environment variable reference.

Runtime dispatch states belong in `WORKFLOW.md` under `tracker.active_states` and `tracker.terminal_states`. Agent-facing labels, role names, and other ticket-writing conventions belong in `docs/agents/triage-labels.md`.

This file documents Linear issue-tracker conventions. It is not the runtime workflow contract. When creating or updating `WORKFLOW.md`, use the `setup-linear-workflow` skill and its Symphony service reference; that workflow skill owns the repository contract shape, while the runner program owns runtime conformance.

## Conventions

- **Create an issue**: create a Linear issue in the configured Linear project.
- **Read an issue**: read the full issue body, state, labels, comments, and relations by Linear identifier, URL, or internal ID.
- **List issues**: filter by project, state, labels, and creation time.
- **Comment on an issue**: create or update comments on the Linear issue.
- **Update state**: use the state requested by the calling skill, `WORKFLOW.md`, or project docs. Do not assume a fixed state list from this template.
- **Update labels**: use the tracker expressions from `docs/agents/triage-labels.md`.
- **Dependencies**: prefer Linear relations that the runner can normalize as `blocked_by`. A `## Blocked by` body section may be added for readability, but it is not a substitute for Linear relations.

## PRD Publishing

When a skill says "publish this PRD to the issue tracker", create a Linear issue in the configured project and use the PRD body as the Linear issue description.

If the project has a PRD label, issue type, or tracker expression, record it in `docs/agents/triage-labels.md` and apply it when publishing. Do not mark PRDs as execution-ready unless the project docs explicitly require it.

If the repository uses Symphony for unattended Linear execution and the user asks for a PRD document rather than a tracker issue, write the PRD to `docs/prd/` instead of creating a Linear issue. Use the `to-prd-doc` skill for that workflow. Implementation issues should link back to that PRD document rather than copying the full PRD into Linear.

Before any implementation issue that links to a PRD document is moved into a Symphony active state, the PRD reference must be reachable by the Symphony workspace. Prefer a PRD file committed to the branch that Symphony workspaces will clone. A local uncommitted PRD file is not enough for active issues.

## Implementation Issue Publishing

When a skill says "publish issues to the issue tracker", create Linear issues in the configured project. Apply canonical triage roles through the tracker expressions in `docs/agents/triage-labels.md`.

Execution eligibility for an unattended runner is controlled by the runtime workflow contract, primarily `WORKFLOW.md` `tracker.active_states`, `tracker.terminal_states`, concurrency settings, and blocker relations. Execution-ready labels are project conventions unless a documented workflow extension implements label filtering.

For repositories that use Symphony for unattended Linear execution, publishing implementation issues is a stateful operation:

1. Read `WORKFLOW.md` before publishing. Confirm `tracker.kind` is `linear`, identify `tracker.active_states`, `tracker.terminal_states`, the target active state for ready AFK issues, and a safe non-active creation state. Prefer `Backlog` for the safe non-active state if it exists.
2. Create all implementation issues in the safe non-active state first. Do not create issues directly in `tracker.active_states`.
3. Create all structured Linear `blocks` relations for dependency edges. If issue B is blocked by issue A, create a Linear relation where A `blocks` B. A `## Blocked by` body section may be included for readability, but it is not a substitute for the structured relation.
4. Read back the created issues and verify each blocked issue exposes the expected runner-normalized relation shape: the blocked issue must have an inverse `blocks` relation from its blocker.
5. Move only unblocked AFK issues to the target Symphony active state, usually `Todo`.
6. Keep every blocked issue and every HITL issue in the safe non-active state. Do not move blocked or HITL issues into `tracker.active_states` automatically. If the user asks for that workflow, report the Symphony runner risk and leave the issue non-active for manual handling.

When reporting success for Symphony-backed Linear publishing, include every created issue identifier, the initial and final state of each issue, every structured `blocks` relation created, which issues are active and Symphony-ready, and which issues were intentionally left non-active.

## When A Skill Says "Publish To The Issue Tracker"

Create a Linear issue and set project, labels, state, and relations according to this file, `docs/agents/triage-labels.md`, and any runtime workflow contract in `WORKFLOW.md`.

If the available Linear tooling can create the issue but cannot set every field, create the issue first and report which fields could not be set automatically.

## When A Skill Says "Fetch The Relevant Ticket"

Fetch the corresponding Linear issue body, comments, labels, state, relations, and URL. Use the Linear identifier as the human-readable reference.

## Linear GraphQL

If `linear_graphql` is available, prefer it for Linear GraphQL operations. Do not require the agent to read Linear tokens from disk; use the configured Linear auth.

`linear_graphql` is an agent tool, not orchestrator business logic. Ticket writes performed through it should follow the workflow prompt and project docs.

## If Linear Writes Are Not Available

If no Linear write-capable tool is available, do not stall. Produce the exact Linear fields and issue body that should be created, report that the issue was not published, and stop.
