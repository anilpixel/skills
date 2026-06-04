---
name: to-prd-doc
description: Use when the user wants to turn the current context into a PRD document for a repository, especially before creating Symphony/Linear implementation issues.
---

# To PRD Doc

This skill takes the current conversation context and codebase understanding and produces a PRD document in the project repository. Do NOT interview the user; synthesize what you already know.

Issue tracker and triage label vocabulary may be useful context, but this skill does not publish a Linear issue. A PRD is a design artifact, not an executable work item.

## Process

1. Explore the repo to understand the current state of the codebase, if you have not already. Use the project's domain glossary vocabulary throughout the PRD, and respect any ADRs in the area you're touching.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can.

Do not interview the user about product requirements. If the testing seams would materially change how implementation issues are split, confirm that one implementation boundary before writing the PRD.

3. Write the PRD using the template below, then save it under the current repository's `docs/prd/` directory.

Use this filename format:

```text
docs/prd/YYYY-MM-DD-short-kebab-title.md
```

If `docs/prd/` does not exist, create it. Do not create or update Linear issues in this skill. Do not put PRDs into the runner's active queue. After writing the file, report the path so follow-up issue publishing can link to it.

Before handing off to issue publishing, make the PRD reference stable for Symphony agents:

- Prefer a PRD file committed to the branch that Symphony workspaces will clone.
- If the PRD is not committed or otherwise reachable by Symphony, report that implementation issues must remain non-active until the PRD is available.

<prd-template>

# <PRD Title>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should cover the feature's important behaviors, boundaries, and roles.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can, such as a state machine, reducer, schema, or type shape, inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts: not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test: only test external behavior, not implementation details
- Which modules or boundaries will be tested
- Prior art for the tests, such as similar tests in the codebase

## Issue Publishing Notes

Guidance for follow-up implementation issue creation. Include:

- Suggested vertical slices
- Which slices are AFK versus HITL
- Any dependency ordering that must become structured Linear `blocks` relations
- Any slices that should not enter the Symphony active queue

## Out of Scope

A description of the things that are out of scope for this PRD.

## Further Notes

Any further notes about the feature.

</prd-template>

## Handoff To Issue Publishing

When implementation issues are needed, use `to-issues` and pass the PRD file path. If the repo uses Linear plus Symphony, issue publishing must follow the Symphony-specific rules in `docs/agents/issue-tracker.md`, generated from `setup-matt-pocock-skills/issue-tracker-linear.md`. Implementation issue bodies should link back to the PRD instead of copying the entire PRD into Linear.
