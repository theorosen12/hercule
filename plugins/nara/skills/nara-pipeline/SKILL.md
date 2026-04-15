---
name: nara-pipeline
description: "Pipeline status dashboard for Nara — scans the active feature folder, reports the state of all 9 phases (0=Linear Context, 1=Brainstorm, 2=PRD, 3=UX, 4=Architecture, 5=Stories, 6=Dev, 7=QA, 8=Release), and tells you exactly which skill to run next. Always start here."
---

# Nara Pipeline Status

**Your Role:** Pipeline navigator. Scan the feature folder, report the state of every phase, and tell the team exactly which skill to invoke next.

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. If the argument provided is a feature slug, use it directly. Otherwise, proceed to step 2.

## EXECUTION

### Step 1 — Identify active feature

Check if a feature slug was provided as an argument. If not, list all folders under `{output_folder}/features/` and ask the user to pick one. If none exist, tell the user to start with `/nara-start [linear-issue-id]`.

Set `{feature_folder}` = `{output_folder}/features/{slug}/`.

### Step 2 — Scan artifacts

Check for existence of each file:

| Artifact | Path |
|----------|------|
| Linear Context | `{feature_folder}/context.md` |
| Brainstorm | `{feature_folder}/brainstorm.md` |
| PRD | `{feature_folder}/prd.md` |
| UX Spec | `{feature_folder}/ux-spec.md` |
| Architecture | `{feature_folder}/architecture.md` |
| Stories | `{feature_folder}/stories/` (any `.md` files) |
| QA Checklist | `{feature_folder}/qa-checklist.md` |
| Release Notes | `{feature_folder}/release-notes.md` |
| Retrospective | `{feature_folder}/retrospective.md` |

### Step 3 — Display status table

```
NARA PIPELINE — {feature_name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Phase 0 · Linear Context  [✅ DONE | ❌ MISSING]
Phase 1 · Brainstorm      [✅ DONE | ❌ MISSING]
Phase 2 · PRD             [✅ DONE | ❌ MISSING]
Phase 3 · UX Spec         [✅ DONE | ❌ MISSING]
Phase 4 · Architecture    [✅ DONE | ❌ MISSING]
Phase 5 · Stories         [✅ N stories | ❌ MISSING]
Phase 6 · Dev             [🔄 N/M done | ✅ DONE | ⏳ NOT STARTED]
Phase 7 · QA              [✅ DONE | ❌ MISSING]
Phase 8 · Release         [✅ DONE | ❌ MISSING]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

For Phase 6, count story files and check each for a `Status: Done` marker.

### Step 4 — Recommend next action

Based on the first incomplete phase, output:

```
▶ NEXT: /nara-[phase] [feature-slug]
```

Include a one-line explanation of what that skill will produce.

**Phase guidance:**
- Phase 0 missing → `/nara-start [issue-id]` — pull feature context from Linear (issue, comments, epic, linked docs)
- Phase 1 missing → `/nara-brainstorm {slug}` — ideation session + Pendo signals, builds on context.md
- Phase 2 missing → `/nara-prd {slug}` — full BMAD PRD (12-step) grounded in Pendo data
- Phase 3 missing → `/nara-ux {slug}` — full BMAD UX design (14-step) + Storybook component mapping
- Phase 4 missing → `/nara-arch {slug}` — Epic-level architecture: ALL decisions made upfront — Prisma schema, exact TypeScript interfaces, GraphQL SDL, complete file tree, edge case handling — zero ambiguity left for implementation
- Phase 5 missing → `/nara-stories {slug}` — stories derived from architecture with verbatim interface excerpts embedded; ambiguity audit + HARD BLOCKING readiness gate + Linear population
- Phase 6 incomplete → `/nara-dev {slug} [story-id]` — pure implementation against story spec; no design decisions; escalates ambiguity; mandatory code review
- Phase 7 missing → `/nara-qa {slug}` — edge case hunt + Maestro E2E tests + QA checklist
- Phase 8 missing → `/nara-release {slug}` — release notes + Pendo setup + Linear closeout + retrospective

If all phases are complete: congratulate the team — the pipeline is fully closed.
