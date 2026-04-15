---
name: nara-stories
description: "Phase 5 of the Nara pipeline — derive implementation-ready stories from architecture.md. Each story embeds verbatim architecture excerpts and leaves zero design decisions for dev. Includes HARD BLOCKING readiness gate and Linear population. Requires architecture.md from /nara-arch."
---

# Nara Stories — Phase 5

**Your Role:** Scrum Master and story translator. Your job is to take the architecture document and break it into stories so precise that a developer can implement each one without making a single design or architectural decision.

**Principle:** A story is implementation-ready when every task references a specific interface, method, file path, or type from `architecture.md`. If a task is vague, it is incomplete.

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. Resolve `{feature_folder}` from argument or ask user.

## EXECUTION

### Step 1 — Verify prerequisites

Check that all three exist: `prd.md`, `ux-spec.md`, `architecture.md`. If any is missing → stop and name the missing phase.

Read all three documents fully before writing a single story.

### Step 2 — Create stories folder

Create `{feature_folder}/stories/` if it doesn't exist.

### Step 3 — Plan the story breakdown

Before writing any story file, produce a story breakdown table for user review:

| Story ID | Title | Layer | Architecture Sections Covered | Depends On |
|----------|-------|-------|-------------------------------|------------|
| S-01 | [e.g., Data Model & Migration] | Backend | §2.1, §2.3 | — |
| S-02 | [e.g., Repository & Service Layer] | Backend | §4.1, §4.2, §5.1 | S-01 |
| S-03 | [e.g., GraphQL Resolver] | Backend | §3.1–3.4, §5.1 | S-02 |
| S-04 | [e.g., Mobile Hook & GraphQL Op] | Mobile | §4.3, §6.3 | S-03 |
| S-05 | [e.g., Screen & Components] | Mobile | §4.4, §6.1, §6.4 | S-04 |

Ask user to confirm or adjust the breakdown before writing story files.

### Step 4 — Create story files

For each story, create `{feature_folder}/stories/{story-id}-{slug}.md`.

Each story must follow this template exactly. Sections marked **[EMBED]** require verbatim excerpts from `architecture.md` — do not paraphrase, copy the relevant content directly into the story.

```markdown
# {STORY-ID} — {Story Title}

**Epic:** {feature_name}
**Status:** Specified
**Priority:** {Low|Medium|High}
**Layer:** {Backend | Mobile | Full-stack}
**Depends on:** {story-id or "none"}

---

## Goal
[One sentence: what concrete artifact this story produces — name exact files or features]

---

## Architectural Context [EMBED]
<!-- Verbatim excerpts from architecture.md — dev must not need to open architecture.md -->

### Data Model (if applicable)
```prisma
// Exact Prisma model block from architecture.md §2
```

### Interfaces & Signatures (if applicable)
```typescript
// Exact TypeScript interfaces from architecture.md §4
```

### GraphQL Types / Operations (if applicable)
```graphql
// Exact SDL or .graphql from architecture.md §3 or §6.3
```

### Navigation (if applicable)
| Route Name | Screen | Params |

---

## Files to Create
| File Path | Class/Function Name | Responsibility |

## Files to Modify
| File Path | Change |

---

## Tasks
<!-- Each task must reference a specific interface, method, type, or file from §Architectural Context -->
<!-- FORBIDDEN: vague tasks like "create the service" -->
<!-- REQUIRED: precise tasks like "implement I{Name}Service.methodName() as specified above" -->
- [ ] [precise, reference-grounded task]

---

## Acceptance Criteria
<!-- Binary-testable, derived from prd.md ACs scoped to this story -->
- [ ] [testable criterion]

---

## Edge Cases & Required Handling
<!-- Verbatim rows from architecture.md §11 that apply to this story -->
| Edge Case | Handling Strategy | User-Facing Message |

---

## Architectural Constraints
- Do NOT rename any interface, method, or type defined in §Architectural Context
- Do NOT add fields to Prisma models beyond those defined in §Data Model
- Do NOT create files not listed in §Files to Create
- Do NOT use `any` type anywhere in this story

---

## Dev Notes
<!-- Nara-specific patterns, exact references to architecture.md -->
- [e.g., Follow the pattern in `backend/src/trips/services/trip.service.ts` for DI]
- [e.g., Use `@CurrentUser()` decorator for auth]
- [e.g., Run `npm run codegen` from `/mobile` after modifying GraphQL operations]
```

### Step 5 — Ambiguity audit

After writing all story files, run this check on each story:

For every task in every story, ask: **"Can dev implement this task without making a design decision?"**

If no → the task is incomplete. Add the missing specification. Do not proceed until all tasks pass this check.

### Step 6 — HARD BLOCKING GATE — Implementation Readiness

Invoke the `bmad-check-implementation-readiness` skill on all four artifacts (prd.md, ux-spec.md, architecture.md, stories/).

**THIS GATE IS MANDATORY. Do not proceed if it fails.**

If readiness check fails:
- List every gap precisely
- Name which phase each gap belongs to
- Ask user: fix gaps and re-run `/nara-stories`, or go back to the relevant phase skill
- 🛑 STOP

If readiness check passes → continue.

### Step 7 — Gate 5

Present readiness report and story list summary.

> **[A] Confirmed — stories are ready for Linear + implementation** | **[F] Fix gaps first**

🛑 HALT until [A].

### Step 8 — Populate Linear

First, get the team using the Linear MCP `list_teams` tool. Ask user to confirm which team if multiple.

Create the **Epic issue**:
- Use Linear MCP `save_issue`
- Title: `[NAR-XXX] {feature_name} — Epic`
- Description: feature objective from PRD

For each story:
- Use Linear MCP `save_issue`
- Title: story title
- Description: full story file content (goal + tasks + ACs + dev notes)
- Parent: epic issue ID
- Labels: `frontend` / `backend` / `fullstack` as appropriate
- Priority: map story priority to Linear priority

Output a table:
```
Stories created in Linear:
| Story ID | Linear Issue | Title |
|----------|-------------|-------|
```

### Step 9 — Hand off

Output:
```
✅ Phase 5 complete — {N} stories created, readiness gate passed, Linear populated.

Stories: {feature_folder}/stories/
Architecture coverage: all §sections traced to stories ✅

▶ NEXT: /nara-dev {slug} [start with story S-01]
```
