---
name: nara-start
description: Phase 0 — Pull feature context from Linear (issue + project + comments + linked issues), produce context.md, and hand off to /nara-brainstorm.
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara Start — Phase 0

**Your Role:** Context importer. Pull everything the team has already documented in Linear for this feature — description, acceptance criteria, discussions, linked issues — and consolidate it into a single `context.md` artifact that feeds the rest of the pipeline.

## EXECUTION

### Step 1 — Load config

Load `{project-root}/_bmad/bmm/config.yaml`. Resolve `output_folder`, `user_name`.

### Step 2 — Identify the Linear issue

Ask the user:

> What is the Linear issue ID for this feature? (e.g. `NAR-356`)

Set `{issue_id}` = provided value.

### Step 3 — Pull issue data from Linear

Using `mcp__claude_ai_Linear__get_issue` with `{issue_id}`, extract:
- Title
- Description
- Status
- Priority
- Assignee(s)
- Labels
- Due date (if any)
- Parent issue / Epic (if any)

### Step 4 — Pull comments & discussion

Using `mcp__claude_ai_Linear__list_comments` on `{issue_id}`, extract all comments.

Filter for signal: decisions, open questions, disagreements, links to external resources (Figma, Notion, etc.), mentions of acceptance criteria or constraints.

Discard pure status updates ("moving to In Progress", etc.).

### Step 5 — Pull related issues

If the issue has a parent (epic), use `mcp__claude_ai_Linear__get_issue` on the parent to get:
- Epic description and goal
- List of sibling issues in the same epic

If the issue belongs to a Linear project, use `mcp__claude_ai_Linear__get_project` to get:
- Project description
- Project milestones / status

### Step 6 — Pull any attached documents

Using `mcp__claude_ai_Linear__get_attachment` or `mcp__claude_ai_Linear__list_documents`, check for any documents attached to the issue or its project (specs, PRDs, briefs, design links).

List them with their URLs/titles — do not fetch external URLs.

### Step 7 — Determine feature slug

Derive a slug from the issue title (e.g. "Trip Completion Score" → `trip-completion-score`).

Ask user to confirm or override:
> Suggested feature slug: `{slug}` — **[C] Confirm** | **[E] Edit**

Set `{feature_folder}` = `{output_folder}/features/{slug}/`. Create folder if it doesn't exist.

### Step 8 — Write context.md

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

### Step 9 — Gate 0

Present the context summary to the user.

> **[A] Context looks complete — proceed to /nara-brainstorm** | **[E] Edit / add missing context manually**

🛑 HALT until [A].

### Step 10 — Hand off

Output:
```
✅ Phase 0 complete — Linear context imported.

Issue: {issue_id}
Feature slug: {slug}
Artifact: {feature_folder}/context.md

▶ NEXT: /nara-brainstorm {slug}
  The brainstorm session will use context.md as its starting point.
```
