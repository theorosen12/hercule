---
name: nara-arch
description: Phase 3 — Epic-level technical architecture for Nara. Makes ALL architectural decisions so implementation is deterministic. Produces exhaustive TypeScript interfaces, Prisma schema, GraphQL SDL, and file tree before any story is written.
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara Architecture — Phase 3

**Your Role:** Senior full-stack architect and sole technical decision-maker for this feature. Your job is to make every architectural decision BEFORE implementation begins. Dev will execute your output — if a decision is not in your architecture document, dev will stop and escalate, not improvise.

**Principle:** The architecture is complete when a developer can implement every story without making a single design decision. If anything is ambiguous, resolve it here.

**Stack context:** React Native (Expo) · NestJS · Prisma · GraphQL (code-first) · PostgreSQL

---

## EXECUTION

### Step 1 — Load config & verify prerequisites

Load config. Resolve `{feature_folder}`.

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

Record every non-obvious technical choice made in this document. This is the authoritative decision record — dev must not re-evaluate these.

| # | Decision | Rationale | Alternatives Rejected |
|---|----------|-----------|----------------------|
| 1 | [e.g., Use optimistic updates for score display] | [why] | [what else was considered] |
| … | … | … | … |

---

## 2. Data Model (Prisma)

### 2.1 New Models

Provide the exact Prisma schema block, copy-paste ready.

```prisma
model ExampleModel {
  id        String   @id @default(cuid())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  // all fields with types, nullability, relations
}
```

### 2.2 Modifications to Existing Models

For each existing model modified:
- Model name and current state (excerpt)
- Exact fields to add/remove/change
- Migration strategy (additive / breaking / backfill required)

### 2.3 Migration Plan

- Migration name: `{timestamp}_describe_change`
- Is it additive-only? [yes/no]
- Backfill required? [yes — script needed | no]
- Breaking change to existing API? [yes — coordinate with X | no]

---

## 3. GraphQL API (Backend)

### 3.1 New Types / DTOs

Exact GraphQL type definitions (SDL-style, for documentation):

```graphql
type ExampleType {
  id: ID!
  fieldName: String!
  optionalField: Int
  nestedObject: OtherType!
}

input CreateExampleInput {
  fieldName: String!
  optionalField: Int
}
```

### 3.2 New Queries

| Query Name | Args (name: Type) | Returns | Auth Required | Description |
|------------|-------------------|---------|---------------|-------------|
| `exampleQuery` | `id: ID!` | `ExampleType!` | Yes — owner only | … |

### 3.3 New Mutations

| Mutation Name | Args (name: Type) | Returns | Auth Required | Description |
|---------------|-------------------|---------|---------------|-------------|
| `createExample` | `input: CreateExampleInput!` | `ExampleType!` | Yes | … |

### 3.4 Error Handling

List all error conditions and how they are surfaced:

| Error Condition | Error Type | HTTP / GQL Code | Message Strategy |
|-----------------|------------|-----------------|-----------------|
| [e.g., Trip not found] | `NotFoundException` | 404 / NOT_FOUND | "Trip not found" |
| … | … | … | … |

---

## 4. TypeScript Interfaces & Service Contracts

This section defines every interface and method signature. Dev must implement exactly these — no renaming, no signature changes.

### 4.1 Repository Interfaces

```typescript
// {domain}/{name}.repository.interface.ts
export interface I{Name}Repository {
  findById(id: string): Promise<ModelName | null>;
  // all methods with exact signatures
}
```

### 4.2 Service Interfaces

```typescript
// {domain}/{name}.service.interface.ts
export interface I{Name}Service {
  methodName(param: ParamType): Promise<ReturnType>;
  // all public methods
}
```

### 4.3 Mobile Hooks

```typescript
// hooks/{hookName}.ts
// Return type
interface Use{HookName}Result {
  data: DataType | null;
  loading: boolean;
  error: ApolloError | undefined;
  // all returned values with types
}

function use{HookName}(param: ParamType): Use{HookName}Result;
```

### 4.4 Component Props Interfaces

For each NEW component required by the UX spec:

```typescript
// components/{ComponentName}/{ComponentName}.tsx
interface {ComponentName}Props {
  propName: PropType;
  optionalProp?: PropType;
  onAction: (param: ParamType) => void;
}
```

---

## 5. Backend File Structure

### 5.1 Files to Create

List every file with its exact path, NestJS class name, and responsibility. Dev must not create files outside this list.

| File Path | Class Name | Responsibility |
|-----------|------------|----------------|
| `backend/src/{domain}/{name}.module.ts` | `{Name}Module` | NestJS module wiring |
| `backend/src/{domain}/services/{name}.service.ts` | `{Name}Service` | [precise responsibility] |
| `backend/src/{domain}/repositories/{name}.repository.ts` | `{Name}Repository` | [precise responsibility] |
| `backend/src/{domain}/resolvers/{name}.resolver.ts` | `{Name}Resolver` | [precise responsibility] |
| `backend/src/{domain}/dto/{name}.dto.ts` | `{Name}Dto` | Input validation |
| `backend/src/{domain}/types/{name}.type.ts` | `{Name}Type` | GraphQL output type |
| … | … | … |

### 5.2 Files to Modify

| File Path | Change Description | Risk |
|-----------|--------------------|------|
| `backend/src/existing/file.ts` | Add method X to existing service | Low |
| … | … | … |

---

## 6. Mobile File Structure

### 6.1 Files to Create

| File Path | Type | Responsibility |
|-----------|------|----------------|
| `mobile/src/domains/{domain}/view/screens/{Name}Screen.tsx` | Screen | [what it renders] |
| `mobile/src/domains/{domain}/view/components/{Name}/{Name}.tsx` | Component | [what it renders] |
| `mobile/src/domains/{domain}/hooks/use{Name}.ts` | Hook | [data/state it manages] |
| `mobile/src/domains/{domain}/graphql/{name}.graphql` | GraphQL operation | [query or mutation] |
| … | … | … |

### 6.2 Files to Modify

| File Path | Change Description |
|-----------|--------------------|
| … | … |

### 6.3 GraphQL Operations (exact)

For each new GraphQL operation in mobile:

```graphql
# {name}.graphql
query {OperationName}($param: Type!) {
  queryName(param: $param) {
    id
    # exact fields selected — no over-fetching, no under-fetching
  }
}
```

### 6.4 Navigation

| Route Name | Screen Component | Params Type | Entry Points |
|------------|-----------------|-------------|--------------|
| `{RouteName}` | `{ScreenName}` | `{ param: type }` | [where navigation.navigate is called] |

---

## 7. State Management

Describe exactly what state lives where:

| State | Location | Type | Persistence |
|-------|----------|------|-------------|
| [e.g., trip score] | Apollo cache | `TripScore` | In-memory |
| [e.g., modal visible] | Local component state | `boolean` | Ephemeral |
| … | … | … | … |

---

## 8. Security

- **Authentication:** [required? which guard? which decorator?]
- **Authorization:** [who can access what? exact rule]
- **PII fields:** [list any personal data — how stored, how exposed]
- **Input validation:** [class-validator rules for each DTO]

---

## 9. Performance

- **N+1 risks:** [list each — mitigation strategy]
- **Pagination:** [which queries — strategy: cursor / offset]
- **Caching:** [Apollo, Redis — what is cached and TTL]
- **Mobile performance:** [list any FlatList, memoization, or lazy-load requirements]

---

## 10. Mobile-Specific

- **Offline behavior:** [what works offline — what fails gracefully — what is blocked]
- **Push notifications:** [trigger events, payloads, deeplink targets]
- **Device permissions:** [which permissions, when requested, fallback if denied]
- **Network resilience:** [retry logic, optimistic updates, conflict resolution]

---

## 11. Edge Cases & Handling

Document EVERY edge case and its exact handling. Dev must not make edge-case decisions.

| Edge Case | Where It Occurs | Handling Strategy | User-Facing Message |
|-----------|-----------------|-------------------|---------------------|
| [e.g., Trip has no photos] | Score calculation | Return score with photos_count = 0, do not fail | "Add photos to improve your score" |
| … | … | … | … |

---

## 12. Testing Requirements

For each service method and resolver, specify what must be unit-tested:

| Unit | Method | Test Cases Required |
|------|--------|---------------------|
| `{Name}Service` | `calculateScore` | [null input, min values, max values, happy path] |
| … | … | … |

```

### Step 4 — Architecture Completeness Check

Before presenting to the user, self-audit the document against this checklist:

- [ ] Every acceptance criterion in `prd.md` is traceable to a section in this architecture
- [ ] Every screen in `ux-spec.md` has a corresponding Screen component in §6.1
- [ ] Every new component in `ux-spec.md` has a Props interface in §4.4
- [ ] No section contains "TBD", "to be determined", or "depends on implementation"
- [ ] Every error condition is listed in §3.4
- [ ] Every edge case from PRD is listed in §11
- [ ] Every file in §5.1 and §6.1 has an unambiguous responsibility
- [ ] Every method in §4.1–4.3 has a complete signature (no `any`, no missing param names)

If any item is unchecked → resolve it before continuing.

### Step 5 — Gate 3

Present the architecture document with the completeness checklist result.

> **[A] Approve Architecture — all decisions made, no ambiguity** | **[R] Request changes**

🛑 HALT until [A]. If [R] → apply changes and re-run completeness check → re-present.

### Step 6 — Update Linear

Attach `architecture.md` to the Linear project using `mcp__claude_ai_Linear__create_document`.

### Step 7 — Hand off

Update `architecture.md` frontmatter: `stepsCompleted: [architecture-written, architecture-approved]`

Output:
```
✅ Phase 3 complete — Architecture approved. All decisions recorded.

Artifact: {feature_folder}/architecture.md
Decisions made: {N} (see §1 Architectural Decision Log)
Files specified: {N} backend + {N} mobile

▶ NEXT: /nara-stories {slug}
  Stories will be derived directly from this architecture. No design decisions will be made during implementation.
```
