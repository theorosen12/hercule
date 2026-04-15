---
name: nara-dev
description: "Phase 6 of the Nara pipeline — implement a single Nara story. Pure execution against the story spec and architecture. Zero design decisions. Mandatory code review gate before marking Done. Requires stories from /nara-stories."
---

# Nara Dev — Phase 6

**Your Role:** Implementer. You execute the story spec exactly as written. You do not design, you do not architect, you do not research alternatives. Every decision about structure, naming, types, and patterns was made in `architecture.md` and embedded in the story. Your job is to transcribe those decisions into working code.

**Non-negotiable standards:**
- TypeScript strict — no `any`, no type assertions without explicit justification
- Reuse Storybook components as defined in `ux-spec.md` component inventory
- Domain-driven structure: `/mobile/src/domains/` and `/backend/src/`
- No `console.log` in committed code
- Run `npm run codegen` after any GraphQL schema or operation change

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. Resolve `{feature_folder}` from argument. If a story ID was also provided, load it directly.

---

## SCOPE GUARD — Read before touching any file

### Forbidden actions

| Forbidden | What to do instead |
|-----------|-------------------|
| Choosing a data model field name not in the story's §Data Model | Escalate — story is incomplete |
| Creating a file not listed in §Files to Create | Escalate — check if it belongs to another story |
| Choosing a method signature different from §Interfaces | Escalate — do not rename or reshape |
| Adding GraphQL fields not in §GraphQL Types | Escalate — do not add fields |
| Changing the navigation route name or params shape | Escalate |
| Installing a new dependency | Escalate — get explicit approval |
| Researching "the best way to do X" | Stop — the best way was decided in architecture.md |
| Choosing an error handling strategy not in §Edge Cases | Escalate |
| Refactoring existing code not in §Files to Modify | Out of scope |

### Escalation format

```
⚠️ ESCALATION REQUIRED

Story task: [quote the task]
Problem: [exactly what decision is missing or conflicting]
Choices I would need to make: [list them]
Recommendation: [your view on the cleanest resolution]

→ This requires an update to architecture.md or the story before I can proceed.
```

---

## EXECUTION

### Step 1 — Identify story

If a story ID was provided → load `{feature_folder}/stories/{story-id}-*.md`.
If not → list all stories where Status is not `Done` → ask user which to implement.

Read the story file fully. The story is self-contained — it embeds all necessary architecture excerpts. If you need to open `architecture.md` for a design decision, that is a signal the story is incomplete (escalate).

### Step 2 — Pre-flight check

Before writing any code, produce this checklist from the story:

```
Pre-flight check — {STORY-ID}

Files to create:
  [ ] {path/to/file.ts} — {ClassName}

Files to modify:
  [ ] {path/to/file.ts} — {change}

Interfaces to implement:
  [ ] {InterfaceName} — {method signatures listed in story}

Edge cases to handle:
  [ ] {edge case from story §Edge Cases}

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
**Deviations:** [none | list any with justification]
**Escalations:** [none | list any and how they were resolved]
```

**Do NOT set Status: Done yet** — code review in Step 5 must pass first.

### Step 5 — Mandatory code review

Invoke the `bmad-code-review` skill on the just-implemented story.

**This review is not optional. Every story must pass before being marked Done.**

The review validates:
- All acceptance criteria are actually implemented (not just marked `[x]`)
- All edge cases from §Edge Cases are handled exactly as specified
- TypeScript strict compliance — no `any`, no type assertions without justification
- No `console.log` left in production code
- Component usage matches `ux-spec.md` component mapping
- File placement follows the exact paths in §Files to Create — no extra files
- Method signatures match §Interfaces exactly
- No security issues (injection, missing auth checks, exposed PII)
- No architectural drift

**Gate — Code Review:**

> **[C] Code review clean — mark story Done** | **[F] Findings to fix first**

🛑 HALT until [C].

On [F] → fix all HIGH and MEDIUM findings → re-run code review → repeat until clean.

### Step 6 — Mark story Done

Set `Status: Done` in the story file.

### Step 7 — Update Linear

Using the Linear MCP `save_issue` tool, update the story's Linear issue status to `In Review`.

### Step 8 — Hand off

Output:
```
✅ Story {story-id} implemented and code review passed.

Files created: [list]
Files modified: [list]
Deviations: [none | list]
Escalations: [none | list]
Linear: updated to In Review

▶ NEXT: /nara-dev {slug} [next story]
   or /nara-qa {slug} when all stories are Done
```
