---
name: nara-qa
description: Phase 6 — Edge case hunt (bmad-review-edge-case-hunter), then Maestro E2E test generation and QA checklist from story acceptance criteria.
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara QA — Phase 6

**Your Role:** QA engineer. Every acceptance criterion becomes a test. Every edge case gets verified. No gaps.

## EXECUTION

### Step 1 — Load config & verify prerequisites

Load config. Resolve `{feature_folder}`.

Check that all stories in `{feature_folder}/stories/` have `Status: Done`. If not → list incomplete stories → tell user to run `/nara-dev` for each remaining story first.

Read all story files, collecting all acceptance criteria and edge cases.
Read `{feature_folder}/prd.md` for the full edge cases list defined during product.

### Step 2 — Edge case hunt (bmad-review-edge-case-hunter)

Before generating tests, run a systematic edge case analysis to catch anything missed during story writing.

Load and follow the `bmad-review-edge-case-hunter` skill with the following inputs:
- All story files in `{feature_folder}/stories/`
- `{feature_folder}/prd.md`
- `{feature_folder}/ux-spec.md`
- `{feature_folder}/architecture.md`

The hunter should walk every branching path and boundary condition, focusing on:
- Offline / poor network scenarios (mobile-critical)
- Empty states and first-use flows
- Permission denied scenarios (push notifications, location, etc.)
- Concurrent user actions
- Data boundary conditions (empty strings, max lengths, null values)
- Navigation edge cases (back navigation mid-flow, deep links)
- Push notification handling when app is in foreground / background / killed
- Error states and recovery paths

Output: a list of **unhandled edge cases** — those not already covered by existing story acceptance criteria.

Ask user:
> {N} unhandled edge cases discovered. How to handle them?
> **[A] Add to QA checklist as manual tests** | **[S] Create new story files for each** | **[I] Ignore (explain why)**

On [S] → create a new story file per edge case in `{feature_folder}/stories/edge-{n}.md` and stop, telling user to run `/nara-dev` for those stories first.
On [A] → incorporate all discovered edge cases into the QA checklist in Step 4.
On [I] → document the rationale in `qa-checklist.md` under a "Deliberately excluded" section.

### Step 3 — Generate Maestro E2E tests

For each acceptance criterion create a Maestro test in `/mobile/maestro/{feature-slug}/`.

Maestro test file format:
```yaml
# {story-id}-{criterion-slug}.yaml
appId: com.nara.app  # use actual app ID from app.config.ts
---
- launchApp
- [steps to reach the feature]
- [assertion for this criterion]
```

Follow patterns from existing tests in `/mobile/maestro/`.

Group tests by story. Name files clearly: `{story-id}-{criterion-number}-{short-description}.yaml`.

For edge cases added in Step 2 that are automatable → also generate Maestro tests for them.

### Step 4 — Write QA Checklist

Create `{feature_folder}/qa-checklist.md`:

```markdown
# QA Checklist — {feature_name}

## Functional Tests
[One row per acceptance criterion]
| Story | Criterion | Test file | Status |
|-------|-----------|-----------|--------|
| ...   | ...       | ...       | ⬜     |

## Edge Cases (from stories)
[One row per edge case defined in PRD/stories]
| Edge case | Expected behaviour | Test file | Status |
|-----------|-------------------|-----------|--------|
| ...       | ...               | ...       | ⬜     |

## Edge Cases (discovered by hunter)
[One row per edge case found in Step 2]
| Edge case | Expected behaviour | Test file / Manual | Status |
|-----------|-------------------|-------------------|--------|
| ...       | ...               | ...               | ⬜     |

## Regression Tests
[Adjacent features that could be affected]
| Feature | Test to run | Status |
|---------|-------------|--------|
| ...     | ...         | ⬜     |

## Manual Tests (not automatable)
| Scenario | Steps | Expected | Status |
|----------|-------|----------|--------|
| ...      | ...   | ...      | ⬜     |

## Deliberately excluded
| Scenario | Rationale |
|----------|-----------|
| ...      | ...       |
```

### Step 5 — Update Linear

Using `mcp__claude_ai_Linear__save_issue`, update all feature stories to status `In QA`.

### Step 6 — Gate 6

Present the QA checklist and list of generated Maestro files.

> Please run the Maestro tests and fill in the checklist Status column before approving.
> `cd mobile && npx maestro test maestro/{feature-slug}/`

> **[A] QA passed — all tests green, checklist complete** | **[F] Bugs found — describe them**

🛑 HALT until [A]. On [F] → create bug descriptions → user runs `/nara-dev` to fix → return to Step 3.

### Step 7 — Hand off

Output:
```
✅ Phase 6 complete — QA passed.

Edge cases hunted: {N} unhandled cases discovered and addressed
Maestro tests: /mobile/maestro/{feature-slug}/
Checklist: {feature_folder}/qa-checklist.md

▶ NEXT: /nara-release {slug}
```
