---
name: nara-prd
description: Phase 1 — Create a data-informed PRD for a Nara feature. Consumes brainstorm.md, runs Pendo data brief, delegates to the full BMAD PRD workflow (12 steps), validates, creates Linear project.
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara PRD — Phase 1

**Your Role:** Product Manager. Produce a complete, unambiguous PRD grounded in Pendo data and the brainstorming decisions from Phase 0. Do not proceed past a gate without explicit user approval.

## EXECUTION

### Step 1 — Load config & resolve paths

Load `{project-root}/_bmad/bmm/config.yaml`. Resolve `output_folder`, `user_name`.

If a feature slug was not provided as an argument, ask:
1. **Feature slug** (e.g. `trip-completion-score`)

Set `{feature_folder}` = `{output_folder}/features/{slug}/`. Create folder if it doesn't exist.

### Step 2 — Check for brainstorm.md

Check if `{feature_folder}/brainstorm.md` exists.

**If it exists:** Read it fully. Announce:
> "Brainstorm.md found — I'll use the decisions and selected approach as the foundation for this PRD."

**If it does NOT exist:** Warn the user:
> ⚠️ No brainstorm.md found for this feature. Phase 0 (brainstorming) is recommended before writing the PRD to ensure the concept is well-explored and decisions are grounded. You can run `/nara-brainstorm {slug}` now.
>
> **[C] Continue without brainstorm** | **[B] Run /nara-brainstorm first**

On [B] → stop and tell user to run `/nara-brainstorm {slug}`.
On [C] → continue to Step 3 and ask user to describe the feature idea directly.

### Step 3 — Pendo data brief

Using the `pendo-analytics` plugin, run:
- `/pendo-analytics:feature-adoption` — adoption of related existing features
- `/pendo-analytics:feedback-analysis` — user feedback themes related to this feature area
- `/pendo-analytics:session-replay` — sessions with friction in this area

Produce `{feature_folder}/data-brief.md`:

```markdown
# Data Brief — {feature_name}

## Pendo Signals
- Feature adoption: [findings]
- Feedback themes: [findings]
- Session friction: [findings]

## Verdict
- Data support: YES / PARTIAL / NO
- Key insight: [1-2 sentences]
- Risk if not built: [1 sentence]
```

Present the Data Brief. Ask:

> Does this feature deserve prioritization now?
> **[P] Proceed** | **[R] Revise idea** | **[D] Deprioritize**

🛑 HALT. On [D] → stop. On [R] → return to Step 2.

### Step 4 — Write PRD using full BMAD workflow

Load and follow `{project-root}/_bmad/bmm/workflows/2-plan-workflows/create-prd/workflow-create-prd.md`.

When loading this workflow, inject the following Nara-specific context:
- **Output file:** `{feature_folder}/prd.md`
- **Pre-loaded context:** contents of `brainstorm.md` (if available) and `data-brief.md`
- **Product context:** Nara is a travel memory mobile app (React Native/Expo + NestJS + Prisma + GraphQL). Users are travelers. Success metrics must be trackable in Pendo.
- **Nara-specific required sections to add to the PRD** (in addition to the standard BMAD PRD template):
  - `## Security Considerations` — Auth, PII, data access rules
  - `## Pendo Tracking Plan` — Which events will measure success metrics
  - `## Mobile-specific constraints` — Offline behaviour, push notifications, device permissions

Follow the full 12-step BMAD PRD creation workflow including all collaborative sections. Do not skip steps.

### Step 5 — Validate PRD

Load and follow `{project-root}/_bmad/bmm/workflows/2-plan-workflows/create-prd/workflow-validate-prd.md` against `{feature_folder}/prd.md`.

Additionally validate Nara-specific requirements:
- Every success metric has a corresponding Pendo event in the Tracking Plan
- Security Considerations section is present and non-empty
- Mobile-specific constraints are addressed

If validation fails → list every gap → ask user to fill them → re-validate.

Update `prd.md` frontmatter: `stepsCompleted: [data-brief, prd-written, prd-validated]`

### Step 6 — Gate 1

Present PRD summary to user.

> **[A] Approve PRD and proceed to /nara-ux** | **[E] Edit PRD**

🛑 HALT until [A].

### Step 7 — Create Linear project

Using `mcp__claude_ai_Linear__save_project`:
- Title: `NAR — {feature_name}`
- Description: PRD objective + success metrics (1 paragraph)

Using `mcp__claude_ai_Linear__create_document`:
- Attach full `prd.md` content to the project

Confirm Linear project URL to user.

### Step 8 — Hand off

Output:
```
✅ Phase 1 complete — PRD approved and saved to Linear.

Artifacts:
  {feature_folder}/data-brief.md
  {feature_folder}/prd.md

Linear: [project URL]

▶ NEXT: /nara-ux {slug}
```
