---
name: nara-stories
description: Phase 4 — Derive implementation-ready stories directly from architecture.md. Each story embeds verbatim architecture excerpts and leaves zero design decisions for dev.
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara Stories — Phase 4

**Your Role:** Scrum Master and story translator. Your job is to take the architecture document and break it into stories so precise that a developer can implement each one without making a single design or architectural decision.

**Principle:** A story is implementation-ready when every task references a specific interface, method, file path, or type from `architecture.md`. If a task is vague, it is incomplete.

---

## EXECUTION

### Step 1 — Load config & verify prerequisites

Load config. Resolve `{feature_folder}`.

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
| … | … | … | … | … |

Ask user to confirm or adjust the breakdown before writing story files.

### Step 4 — Create story files

For each story, create `{feature_folder}/stories/{story-id}-{slug}.md`.

Each story must follow this template exactly. Sections marked **[EMBED]** require verbatim excerpts from `architecture.md` — do not paraphrase, do not use "see architecture.md §X", copy the relevant content directly into the story.

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

<!-- Copy the relevant excerpt from architecture.md §1 Architectural Decision Log that applies to this story -->
<!-- Copy the relevant type definitions, interfaces, or schema blocks from architecture.md §2–§7 -->
<!-- This section must be self-contained — dev must not need to open architecture.md to implement this story -->

### Data Model (if applicable)
```prisma
// Exact Prisma model block from architecture.md §2
```

### Interfaces & Signatures (if applicable)
```typescript
// Exact TypeScript interfaces from architecture.md §4
// Dev implements these — no changes to signatures allowed
```

### GraphQL Types / Operations (if applicable)
```graphql
// Exact SDL or .graphql operation from architecture.md §3 or §6.3
```

### Navigation (if applicable)
| Route Name | Screen | Params |
|------------|--------|--------|
<!-- From architecture.md §6.4 -->

---

## Files to Create

<!-- Exact rows from architecture.md §5.1 or §6.1 that this story is responsible for -->
| File Path | Class/Function Name | Responsibility |
|-----------|---------------------|----------------|
| `path/to/file.ts` | `ClassName` | [from architecture.md] |

## Files to Modify

<!-- Exact rows from architecture.md §5.2 or §6.2 that this story touches -->
| File Path | Change |
|-----------|--------|
| `path/to/file.ts` | [exact change] |

---

## Tasks

<!-- Each task must reference a specific interface, method, type, or file from the Architectural Context above -->
<!-- FORBIDDEN: vague tasks like "create the service" or "add the query" -->
<!-- REQUIRED: precise tasks like "implement I{Name}Service.methodName() as specified in §Interfaces above" -->

- [ ] [e.g., Add `ExampleModel` to `backend/prisma/schema.prisma` exactly as defined in §Data Model above]
- [ ] [e.g., Run `npx prisma migrate dev --name {migration_name}` to apply the migration]
- [ ] [e.g., Implement `I{Name}Repository` interface in `backend/src/{domain}/repositories/{name}.repository.ts`]
- [ ] [e.g., Implement `I{Name}Service.calculateScore(tripId: string): Promise<TripScore>` in `backend/src/{domain}/services/{name}.service.ts`]
- [ ] [e.g., Create `{Name}Resolver` at `backend/src/{domain}/resolvers/{name}.resolver.ts` — expose `{queryName}` query as defined in §GraphQL above]
- [ ] [...]

---

## Acceptance Criteria

<!-- Derived directly from prd.md ACs scoped to this story — make them binary-testable -->
- [ ] [e.g., `calculateScore()` returns a `TripScore` object with all 5 criteria fields populated]
- [ ] [e.g., GraphQL query `{queryName}` returns 404 when trip does not belong to authenticated user]
- [ ] [...]

---

## Edge Cases & Required Handling

<!-- Verbatim rows from architecture.md §11 that apply to this story — dev must implement ALL of these -->
| Edge Case | Handling Strategy | User-Facing Message |
|-----------|-------------------|---------------------|
| [from arch §11] | [from arch §11] | [from arch §11] |

---

## Architectural Constraints

<!-- Hard rules that dev must NOT deviate from in this story -->
- Do NOT rename any interface, method, or type defined in §Architectural Context
- Do NOT add fields to Prisma models beyond those defined in §Data Model
- Do NOT create files not listed in §Files to Create
- Do NOT use `any` type anywhere in this story
- [Any story-specific constraints from architecture.md §1 Decision Log]

---

## Dev Notes

<!-- Nara-specific patterns to follow, extracted from architecture.md or codebase exploration -->
- [e.g., Follow the pattern in `backend/src/trips/services/trip.service.ts` for dependency injection]
- [e.g., Use `@CurrentUser()` decorator for auth — see `backend/src/auth/decorators/`]
- [e.g., Run `npm run codegen` from `/mobile` after modifying GraphQL operations]
```

### Step 5 — Ambiguity audit

After writing all story files, run this check on each story:

For every task in every story, ask: **"Can dev implement this task without making a design decision?"**

If no → the task is incomplete. Add the missing specification (exact interface, exact method signature, exact file path, exact type). Do not proceed until all tasks pass this check.

### Step 6 — HARD BLOCKING GATE — Implementation Readiness

Load and follow `{project-root}/_bmad/bmm/workflows/3-solutioning/check-implementation-readiness/workflow.md`.

Input: all four artifacts (prd.md, ux-spec.md, architecture.md, stories/).

**THIS GATE IS MANDATORY. Do not proceed if it fails.**

If readiness check fails:
- List every gap precisely
- Name which phase each gap belongs to (usually arch if interfaces are missing, stories if tasks are vague)
- Ask user: fix gaps and re-run `/nara-stories`, or go back to the relevant phase skill
- 🛑 STOP

If readiness check passes → continue.

### Step 7 — Gate 4

Present readiness report and story list summary.

> **[A] Confirmed — stories are ready for Linear + implementation** | **[F] Fix gaps first**

🛑 HALT until [A].

### Step 8 — Populate Linear

First, get the team using `mcp__claude_ai_Linear__list_teams`. Ask user to confirm which team if multiple.

Create the **Epic issue**:
- `mcp__claude_ai_Linear__save_issue`
- Title: `[NAR-XXX] {feature_name} — Epic`
- Description: feature objective from PRD

For each story:
- `mcp__claude_ai_Linear__save_issue`
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
| ...      | ...         | ...   |
```

### Step 9 — Hand off

Output:
```
✅ Phase 4 complete — {N} stories created, readiness gate passed, Linear populated.

Stories: {feature_folder}/stories/
Architecture coverage: all §sections traced to stories ✅

▶ NEXT: /nara-dev {slug} [start with story S-01]
  Dev will implement from story specs only. No design decisions will be made during implementation.
```
