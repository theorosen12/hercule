---
name: nara-start
description: "Phase 0 of the Nara pipeline — pull feature context from a Linear issue (description, comments, epic, linked docs) and produce context.md before brainstorming. Use when the user says 'nara start [issue-id]' or 'start [feature]' or after /nara-pipeline says Phase 0 is next."
---

# Nara Start — Phase 0

**Your Role:** Context importer. Pull everything the team has already documented in Linear for this feature — description, acceptance criteria, discussions, linked issues — and consolidate it into a single `context.md` artifact that feeds the rest of the pipeline.

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. If a Linear issue ID was provided as an argument, use it directly (skip Step 1 prompt).

## EXECUTION

### Step 1 — Identify the Linear issue

If no issue ID was provided, ask the user:

> What is the Linear issue ID for this feature? (e.g. `NAR-356`)

Set `{issue_id}` = provided value.

### Step 2 — Pull issue data from Linear

Using the Linear MCP tool `get_issue` with `{issue_id}`, extract:
- Title
- Description
- Status
- Priority
- Assignee(s)
- Labels
- Due date (if any)
- Parent issue / Epic (if any)

### Step 3 — Pull comments & discussion

Using the Linear MCP tool `list_comments` on `{issue_id}`, extract all comments.

Filter for signal: decisions, open questions, disagreements, links to external resources (Figma, Notion, etc.), mentions of acceptance criteria or constraints.

Discard pure status updates ("moving to In Progress", etc.).

### Step 4 — Pull related issues

If the issue has a parent (epic), use `get_issue` on the parent to get:
- Epic description and goal
- List of sibling issues in the same epic

If the issue belongs to a Linear project, use `get_project` to get:
- Project description
- Project milestones / status

### Step 5 — Pull any attached documents

Check for any documents attached to the issue or its project (specs, PRDs, briefs, design links). List them with their URLs/titles — do not fetch external URLs.

### Step 6 — Determine feature slug

Derive a slug from the issue title (e.g. "Trip Completion Score" → `trip-completion-score`).

Ask user to confirm or override:
> Suggested feature slug: `{slug}` — **[C] Confirm** | **[E] Edit**

Set `{feature_folder}` = `{output_folder}/features/{slug}/`. Create folder if it doesn't exist.

### Step 7 — Write context.md

Create `{feature_folder}/context.md`:

```markdown
# Linear Context — {feature_name}

## Source
- Issue: {issue_id} — {issue_title}
- Status: {status} | Priority: {priority}
- Assignee(s): {assignees}
- Labels: {labels}

## Epic / Project
- Epic: {parent_issue_id} — {parent_title} (if applicable)
- Project: {project_name} (if applicable)

## Feature Description
{issue_description — full, verbatim}

## Key Decisions & Constraints (from comments)
{extracted signal from comments — bullet points}

## Open Questions (from comments)
{unresolved questions mentioned in the thread}

## Linked Resources
{list of attached documents, Figma links, Notion pages, etc.}

## Sibling Issues (same epic)
| Issue ID | Title | Status |
|----------|-------|--------|
| ...      | ...   | ...    |

## Raw Notes
{anything else from the issue that doesn't fit above but could be useful}
```

### Step 8 — Gate 0

Present the context summary to the user.

> **[A] Context looks complete — proceed to /nara-brainstorm** | **[E] Edit / add missing context manually**

🛑 HALT until [A].

### Step 9 — Hand off

Output:
```
✅ Phase 0 complete — Linear context imported.

Issue: {issue_id}
Feature slug: {slug}
Artifact: {feature_folder}/context.md

▶ NEXT: /nara-brainstorm {slug}
  The brainstorm session will use context.md as its starting point.
```
