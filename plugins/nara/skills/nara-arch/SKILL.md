---
name: nara-arch
description: "Phase 4 of the Nara pipeline — epic-level technical architecture. Makes ALL decisions before implementation: Prisma schema, TypeScript interfaces, GraphQL SDL, file tree, edge cases. Zero ambiguity left for dev. Requires prd.md and ux-spec.md."
---

# Nara Architecture — Phase 4

**Your Role:** Senior full-stack architect and sole technical decision-maker for this feature. Your job is to make every architectural decision BEFORE implementation begins. Dev will execute your output — if a decision is not in your architecture document, dev will stop and escalate, not improvise.

**Principle:** The architecture is complete when a developer can implement every story without making a single design decision. If anything is ambiguous, resolve it here.

**Stack context:** React Native (Expo) · NestJS · Prisma · GraphQL (code-first) · PostgreSQL

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. Resolve `{feature_folder}` from argument or ask user.

## EXECUTION

### Step 1 — Verify prerequisites

Check that both `{feature_folder}/prd.md` and `{feature_folder}/ux-spec.md` exist. If either is missing → stop and tell user which phase is incomplete.

Read both documents fully. Extract:
- All user-facing features and flows from the PRD
- All screens, components, states from the UX spec
- All acceptance criteria (these must all be traceable to an architectural decision)
- The full scope of the Epic (all stories that will be created)

### Step 2 — Explore existing codebase

Read the following to understand existing patterns:
- `/backend/src/` — all existing services, resolvers, repositories (domain structure, naming, patterns)
- `/backend/prisma/schema.prisma` — current full data model
- `/mobile/src/domains/` — existing mobile domain structure, hooks, GraphQL operations

**Goal:** Identify every existing pattern, type, and convention this feature must follow. Document any conflicts between PRD requirements and existing patterns — resolve them now.

### Step 3 — Write Architecture Document

Create `{feature_folder}/architecture.md`. Work section by section. Ask for clarification before writing any section where requirements are ambiguous. Do not write "TBD" — every open question must be resolved before this document is done.

```markdown
# Architecture — {feature_name}

_Generated: {date} | Epic: {issue_id} | Status: Draft_

---

## 1. Architectural Decision Log

| # | Decision | Rationale | Alternatives Rejected |
|---|----------|-----------|----------------------|

---

## 2. Data Model (Prisma)

### 2.1 New Models
[exact Prisma schema blocks, copy-paste ready]

### 2.2 Modifications to Existing Models
[exact fields to add/remove/change per model]

### 2.3 Migration Plan
- Migration name: `{timestamp}_describe_change`
- Additive-only? [yes/no]
- Backfill required? [yes — script needed | no]
- Breaking change to existing API? [yes — coordinate | no]

---

## 3. GraphQL API (Backend)

### 3.1 New Types / DTOs
[exact GraphQL type definitions in SDL style]

### 3.2 New Queries
| Query Name | Args | Returns | Auth Required | Description |

### 3.3 New Mutations
| Mutation Name | Args | Returns | Auth Required | Description |

### 3.4 Error Handling
| Error Condition | Error Type | GQL Code | Message Strategy |

---

## 4. TypeScript Interfaces & Service Contracts

### 4.1 Repository Interfaces
[exact interface with all method signatures]

### 4.2 Service Interfaces
[exact interface with all public method signatures]

### 4.3 Mobile Hooks
[exact hook function signatures and return types]

### 4.4 Component Props Interfaces
[exact Props interface for every new component]

---

## 5. Backend File Structure

### 5.1 Files to Create
| File Path | Class Name | Responsibility |

### 5.2 Files to Modify
| File Path | Change Description | Risk |

---

## 6. Mobile File Structure

### 6.1 Files to Create
| File Path | Type | Responsibility |

### 6.2 Files to Modify
| File Path | Change Description |

### 6.3 GraphQL Operations (exact)
[exact .graphql file content for every operation]

### 6.4 Navigation
| Route Name | Screen Component | Params Type | Entry Points |

---

## 7. State Management
| State | Location | Type | Persistence |

---

## 8. Security
- Authentication: [which guard, which decorator]
- Authorization: [exact rules]
- PII fields: [list + handling]
- Input validation: [class-validator rules per DTO]

---

## 9. Performance
- N+1 risks: [list + mitigation]
- Pagination: [strategy per query]
- Caching: [Apollo / Redis — what, TTL]
- Mobile: [FlatList, memoization, lazy-load requirements]

---

## 10. Mobile-Specific
- Offline behavior: [what works / fails gracefully / is blocked]
- Push notifications: [trigger events, payloads, deeplink targets]
- Device permissions: [which, when, fallback]
- Network resilience: [retry, optimistic updates, conflict resolution]

---

## 11. Edge Cases & Handling
| Edge Case | Where It Occurs | Handling Strategy | User-Facing Message |

---

## 12. Testing Requirements
| Unit | Method | Test Cases Required |
```

### Step 4 — Architecture Completeness Check

Before presenting to the user, self-audit against this checklist:

- [ ] Every acceptance criterion in `prd.md` is traceable to a section in this architecture
- [ ] Every screen in `ux-spec.md` has a corresponding Screen component in §6.1
- [ ] Every new component in `ux-spec.md` has a Props interface in §4.4
- [ ] No section contains "TBD", "to be determined", or "depends on implementation"
- [ ] Every error condition is listed in §3.4
- [ ] Every edge case from PRD is listed in §11
- [ ] Every file in §5.1 and §6.1 has an unambiguous responsibility
- [ ] Every method in §4.1–4.3 has a complete signature (no `any`, no missing param names)

If any item is unchecked → resolve it before continuing.

### Step 5 — Gate 4

Present the architecture document with the completeness checklist result.

> **[A] Approve Architecture — all decisions made, no ambiguity** | **[R] Request changes**

🛑 HALT until [A]. If [R] → apply changes and re-run completeness check → re-present.

### Step 6 — Update Linear

Attach `architecture.md` to the Linear project using the Linear MCP `create_document` tool.

### Step 7 — Hand off

Output:
```
✅ Phase 4 complete — Architecture approved. All decisions recorded.

Artifact: {feature_folder}/architecture.md
Decisions made: {N} (see §1 Architectural Decision Log)
Files specified: {N} backend + {N} mobile

▶ NEXT: /nara-stories {slug}
  Stories will be derived directly from this architecture. No design decisions will be made during implementation.
```
