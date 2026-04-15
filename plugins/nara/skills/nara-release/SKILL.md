---
name: nara-release
description: "Phase 8 of the Nara pipeline — release notes, Pendo tracking setup, Linear closeout, and full BMAD retrospective. Closes the pipeline. Requires qa-checklist.md from /nara-qa."
---

# Nara Release — Phase 8

**Your Role:** Release manager and product analyst. Close the feature cleanly, measure what matters, and extract team learnings with a full retrospective.

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. Resolve `{feature_folder}` from argument or ask user.

## EXECUTION

### Step 1 — Verify prerequisites

Check that `{feature_folder}/qa-checklist.md` exists and all stories have `Status: Done`. If not → stop and tell user.

Read `prd.md` (for success metrics), all story files (for change list), `brainstorm.md` if it exists (for original intent comparison).

### Step 2 — Generate Release Notes

Create `{feature_folder}/release-notes.md`:

```markdown
# Release Notes — {feature_name}

## What's new
[User-facing description of the feature — no technical jargon]

## Changes
- Backend: [list of backend changes]
- Mobile: [list of mobile changes]
- Database: [any schema migrations]

## Known limitations
[Anything deferred, partial, or known to be imperfect]

## Rollback notes
[How to revert if needed]
```

### Step 3 — Configure Pendo tracking

Using the `pendo-analytics` plugin:

1. Map each success metric from `prd.md` to a Pendo event
2. Verify that instrumentation calls exist in the implementation for each key user action
3. Define the Pendo funnel / report structure for this feature

Produce `{feature_folder}/pendo-tracking.md`:
```markdown
# Pendo Tracking — {feature_name}

## Events to track
| Event name | Trigger | Maps to metric |
|-----------|---------|----------------|

## Funnel definition
[Description of the Pendo funnel to create]

## Setup status
- [ ] Events firing in staging
- [ ] Funnel created in Pendo
- [ ] Baseline measurement started
```

### Step 4 — Close Linear

For each story using the Linear MCP `save_issue` tool: set status to `Done`.

Add a comment to the Linear project with release notes summary.

### Step 5 — Gate 8

Present the release summary to the user.

> **[A] Release confirmed — proceed to retrospective** | **[H] Hold — something is not ready**

🛑 HALT until [A].

### Step 6 — Full retrospective

Invoke the `bmad-retrospective` skill with the following Nara-specific inputs:
- **Epic artifacts:** all files in `{feature_folder}/`
- **Original intent:** `brainstorm.md` → "Decisions for the PRD" section (compare what was planned vs what shipped)
- **Pipeline-specific retrospective questions** (in addition to the standard BMAD retro):
  - Was Phase 1 (brainstorming) valuable? Did it change the direction of the feature?
  - Was the Pendo data brief accurate? Did data support the feature as built?
  - Which gate was the hardest to pass, and why?
  - Did edge cases surface during QA that were missed in the PRD or stories?
  - Did the 14-step BMAD UX design produce a better spec than a shorter process would have?
  - Were any code review findings surprising given the story specs?

The retrospective output is saved to `{feature_folder}/retrospective.md`.

### Step 7 — Final summary

Output:
```
✅ PIPELINE COMPLETE — {feature_name}

Artifacts:
  _bmad-output/features/{slug}/
  ├── context.md
  ├── brainstorm.md
  ├── research.md         (if Phase 1 research was run)
  ├── data-brief.md
  ├── prd.md
  ├── ux-spec.md
  ├── architecture.md
  ├── stories/
  ├── qa-checklist.md
  ├── release-notes.md
  ├── pendo-tracking.md
  └── retrospective.md

Linear: project closed ✅
Pendo: {N} events configured ✅
Retrospective: complete ✅
```
