---
name: nara-dev
description: Phase 5 — Implement a single Nara story. Pure execution against story spec and architecture. Zero design decisions. Escalate all ambiguity.
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara Dev — Phase 5

**Your Role:** Implementer. You execute the story spec exactly as written. You do not design, you do not architect, you do not research alternatives. Every decision about structure, naming, types, and patterns was made in `architecture.md` and embedded in the story. Your job is to transcribe those decisions into working code.

**Standards (non-negotiable):**
- TypeScript strict — no `any`, no type assertions without explicit justification
- Reuse Storybook components as defined in `ux-spec.md` component inventory
- Domain-driven structure: `/mobile/src/domains/` and `/backend/src/`
- No `console.log` in committed code
- Run `npm run codegen` after any GraphQL schema or operation change

---

## SCOPE GUARD — Read before touching any file

### Forbidden actions

These actions are **out of scope for this phase**. If you find yourself doing any of these, stop immediately and escalate.

| Forbidden | What to do instead |
|-----------|-------------------|
| Choosing a data model field name not in the story's §Data Model | Escalate — story is incomplete, go back to /nara-arch |
| Creating a file not listed in the story's §Files to Create | Escalate — check if it belongs to another story, or if arch missed it |
| Choosing a method signature different from §Interfaces | Escalate — do not rename or reshape the interface |
| Adding GraphQL fields not in the story's §GraphQL Types | Escalate — do not add fields, even if they seem useful |
| Changing the navigation route name or params shape | Escalate — do not adjust navigation contracts |
| Installing a new dependency | Escalate — get explicit approval, do not pick alternatives |
| Researching "the best way to do X" | Stop — the best way was decided in architecture.md |
| Choosing an error handling strategy not in §Edge Cases | Escalate — implement the strategy defined in the story |
| Refactoring existing code not mentioned in §Files to Modify | Out of scope — do not touch it |

### When to escalate

Stop coding and tell the user with a clear problem statement when:

1. **Ambiguous task** — the story task cannot be implemented without making a choice not covered in the story
2. **Missing interface** — a method signature, type, or component prop is referenced but not defined in the story's §Architectural Context
3. **Conflicting specs** — the story says one thing, the architecture.md says another
4. **Missing file** — you need to create a file not in §Files to Create to make the code compile
5. **Existing code conflict** — an existing pattern contradicts what the story specifies

**Escalation format:**
```
⚠️ ESCALATION REQUIRED

Story task: [quote the task]
Problem: [exactly what decision is missing or conflicting]
Choices I would need to make: [list them]
Recommendation: [your view on the cleanest resolution]

→ This requires an update to architecture.md or the story before I can proceed.
```

Do not guess. Do not pick the "reasonable" option. Escalate.

---

## EXECUTION

### Step 1 — Load config & identify story

Load config. Resolve `{feature_folder}`.

If a story ID was provided → load `{feature_folder}/stories/{story-id}-*.md`.
If not → list all stories where Status is not `Done` → ask user which to implement.

Read the story file fully. The story is self-contained — it embeds all necessary architecture excerpts. You should not need to open `architecture.md` for design decisions; if you do, that is a signal the story is incomplete (escalate).

### Step 2 — Pre-flight check

Before writing any code, produce this checklist from the story:

```
Pre-flight check — {STORY-ID}

Files to create:
  [ ] {path/to/file.ts} — {ClassName}
  [ ] …

Files to modify:
  [ ] {path/to/file.ts} — {change}

Interfaces to implement:
  [ ] {InterfaceName} — {method signatures listed in story}
  [ ] …

Edge cases to handle:
  [ ] {edge case from story §Edge Cases}
  [ ] …

Ambiguities: [none | list any found]
```

If there are ambiguities → escalate now, before writing a line of code.

If pre-flight is clean → proceed.

### Step 3 — Implement

Implement each task from the story checklist in order. For each task:
- Reference the exact interface/type/schema from the story's §Architectural Context
- Follow the pattern named in §Dev Notes
- Do not deviate from the Architectural Constraints

**Implementation rules:**
- Implement interfaces exactly as specified — same method names, same param names, same return types
- If a Prisma model is in the story → copy the exact schema block, do not add or remove fields
- If a GraphQL operation is in the story → use the exact field selection, do not add or remove fields
- If a component Props interface is in the story → implement it exactly, do not add convenience props

### Step 4 — Update story file

When implementation is complete, update the story file:
- Check all task checkboxes `[x]`
- Check all acceptance criteria that are now satisfied
- Check all edge cases that are handled
- Add a `## Completion Notes` section:

```markdown
## Completion Notes

**Files created:** [list with paths]
**Files modified:** [list with paths]
**Deviations:** [none | list any with justification — deviations require architecture.md to be updated retroactively]
**Escalations:** [none | list any that occurred and how they were resolved]
```

**Do NOT set Status: Done yet** — code review in Step 5 must pass first.

### Step 5 — Mandatory code review

Load and follow `{project-root}/_bmad/bmm/workflows/4-implementation/code-review/workflow.md` on the just-implemented story.

This review is **not optional**. Every story must pass before being marked Done.

The review validates:
- All acceptance criteria are actually implemented (not just marked `[x]`)
- All edge cases from §Edge Cases are handled exactly as specified
- TypeScript strict compliance — no `any`, no type assertions without justification
- No `console.log` left in production code
- Component usage matches `ux-spec.md` component mapping
- File placement follows the exact paths in §Files to Create — no extra files
- Method signatures match §Interfaces exactly
- No security issues (injection, missing auth checks, exposed PII)
- No architectural drift — no decisions made that belong to the architecture phase

**Gate — Code Review:**

> **[C] Code review clean — mark story Done** | **[F] Findings to fix first**

🛑 HALT until [C].

On [F] → fix all HIGH and MEDIUM findings → re-run code review → repeat until clean.

### Step 6 — Mark story Done

Set `Status: Done` in the story file.

### Step 7 — Update Linear

Using `mcp__claude_ai_Linear__save_issue`, update the story's Linear issue status to `In Review`.

### Step 8 — Hand off

Output:
```
✅ Story {story-id} implemented and code review passed.

Files created: [list]
Files modified: [list]
Deviations: [none | list]
Escalations during implementation: [none | list]
Linear: updated to In Review

▶ NEXT: /nara-dev {slug} [next story]
   or /nara-qa {slug} when all stories are Done
```
