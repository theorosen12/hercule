---
name: nara-prd
description: "Phase 2 of the Nara pipeline — create a data-informed PRD. Consumes brainstorm.md, runs a Pendo data brief, runs the full BMAD 12-step PRD workflow, validates, and creates a Linear project. Requires brainstorm.md from /nara-brainstorm."
---

# Nara PRD — Phase 2

**Your Role:** Product Manager. Produce a complete, unambiguous PRD grounded in Pendo data and the brainstorming decisions from Phase 1. Do not proceed past a gate without explicit user approval.

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. If a feature slug was not provided as an argument, ask for it. Set `{feature_folder}` = `{output_folder}/features/{slug}/`.

## EXECUTION

### Step 1 — Check for brainstorm.md

Check if `{feature_folder}/brainstorm.md` exists.

**If it exists:** Read it fully. Announce:
> "Brainstorm.md found — I'll use the decisions and selected approach as the foundation for this PRD."

**If it does NOT exist:** Warn the user:
> ⚠️ No brainstorm.md found for this feature. Phase 1 (brainstorming) is recommended before writing the PRD to ensure the concept is well-explored and decisions are grounded. You can run `/nara-brainstorm {slug}` now.
>
> **[C] Continue without brainstorm** | **[B] Run /nara-brainstorm first**

On [B] → stop and tell user to run `/nara-brainstorm {slug}`.
On [C] → continue to Step 2 and ask user to describe the feature idea directly.

### Step 2 — Pendo data brief

Using the `pendo-analytics` plugin, run:
- `pendo-analytics:feature-adoption` — adoption of related existing features
- `pendo-analytics:feedback-analysis` — user feedback themes related to this feature area
- `pendo-analytics:session-replay` — sessions with friction in this area

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

🛑 HALT. On [D] → stop. On [R] → return to Step 1.

### Step 3 — Write PRD using the BMAD create-prd skill

Invoke the `bmad-create-prd` skill with the following Nara-specific context injected:
- **Output file:** `{feature_folder}/prd.md`
- **Pre-loaded context:** contents of `brainstorm.md` (if available) and `data-brief.md`
- **Product context:** Nara is a travel memory mobile app (React Native/Expo + NestJS + Prisma + GraphQL). Users are travelers. Success metrics must be trackable in Pendo.
- **Nara-specific required sections** (in addition to the standard BMAD PRD template):
  - `## Security Considerations` — Auth, PII, data access rules
  - `## Pendo Tracking Plan` — Which events will measure success metrics
  - `## Mobile-specific constraints` — Offline behaviour, push notifications, device permissions

Follow the full 12-step BMAD PRD creation workflow including all collaborative sections. Do not skip steps.

### Step 4 — Validate PRD

Invoke the `bmad-validate-prd` skill against `{feature_folder}/prd.md`.

Additionally validate Nara-specific requirements:
- Every success metric has a corresponding Pendo event in the Tracking Plan
- Security Considerations section is present and non-empty
- Mobile-specific constraints are addressed

If validation fails → list every gap → ask user to fill them → re-validate.

### Step 5 — Gate 2

Present PRD summary to user.

> **[A] Approve PRD and proceed to /nara-ux** | **[E] Edit PRD**

🛑 HALT until [A].

### Step 6 — Create Linear project

Using the Linear MCP tool `save_project`:
- Title: `NAR — {feature_name}`
- Description: PRD objective + success metrics (1 paragraph)

Using the Linear MCP tool `create_document`:
- Attach full `prd.md` content to the project

Confirm Linear project URL to user.

### Step 7 — Hand off

Output:
```
✅ Phase 2 complete — PRD approved and saved to Linear.

Artifacts:
  {feature_folder}/data-brief.md
  {feature_folder}/prd.md

Linear: [project URL]

▶ NEXT: /nara-ux {slug}
```
