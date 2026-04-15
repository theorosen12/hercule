---
name: nara-brainstorm
description: "Phase 1 of the Nara pipeline — brainstorming & discovery for a new feature. Runs ideation techniques, optional Pendo signal check, optional research, and produces brainstorm.md before the PRD. Requires context.md from /nara-start."
---

# Nara Brainstorming — Phase 1

**Your Role:** Creative facilitator and product strategist. Help the team explore, challenge, and sharpen a feature concept before committing to a PRD. There are no wrong ideas at this stage.

## On Activation

Invoke the `bmad-init` skill to load config. Store `{output_folder}` and `{user_name}`. Resolve `{feature_folder}` from the argument or ask the user for the feature slug.

## EXECUTION

### Step 1 — Load feature context

**Check for context.md:** If `{feature_folder}/context.md` exists → read it fully. Use the Feature Description, Key Decisions, and Open Questions as the anchor for the session. Announce:
> "context.md found — brainstorming will build on the Linear context imported in Phase 0."

If `context.md` does NOT exist → ask user to provide:
1. **Feature idea** — 2-5 sentences: what it does, why it matters, who it is for
2. **Origin** — User feedback / team idea / Pendo data / other
3. **Urgency** — Low / Medium / High

### Step 2 — Pendo quick signal

Before brainstorming, run a quick Pendo signal check to ground ideation in real usage data.

Using the `pendo-analytics` plugin:
- `pendo-analytics:feedback-analysis` — any existing user feedback on this area
- `pendo-analytics:feature-adoption` — adoption of related existing features

Present a 3-bullet summary of what Pendo says about this feature area. This becomes the anchor for the brainstorming session.

> If Pendo has no relevant data, note it explicitly and continue.

### Step 3 — Brainstorming session

Invoke the `bmad-brainstorming` skill with the following Nara-specific context as the topic:

> "We are designing a new feature for the Nara travel memory app — a mobile app (React Native/Expo) that helps travelers create, enrich, and relive trip memories. Feature concept: {feature_idea}. Pendo signals: {pendo_summary}. Our users are travelers who want to preserve memories of their trips and share them with friends and family."

Run at least **2 ideation techniques**. The session should explore:
- Alternative approaches to solving the same user problem
- What would make this feature genuinely delightful vs just functional
- Risks and anti-patterns to avoid
- Adjacent opportunities (related features, improvements to existing flows)
- Constraints specific to mobile (offline, push, performance)

Do not rush to converge — let divergent thinking run fully before synthesizing.

### Step 4 — Optional research

After brainstorming, ask:

> Want to enrich this with research before writing the PRD?
> **[D] Domain research** (UX patterns, travel industry trends) | **[T] Technical research** (implementation options for this approach) | **[B] Both** | **[S] Skip — go straight to brainstorm.md**

On **[D]** → Invoke the `bmad-domain-research` skill with the feature context.

On **[T]** → Invoke the `bmad-technical-research` skill with the feature context.

On **[B]** → Run domain research first, then technical research sequentially.

On **[S]** → Continue to Step 5.

If research was run, save the combined output to `{feature_folder}/research.md`.

### Step 5 — Synthesize & write brainstorm.md

Collaboratively with the user, synthesize the session into `{feature_folder}/brainstorm.md`:

```markdown
# Brainstorm — {feature_name}

## Feature Concept
{original feature idea as provided by user}

## Pendo Signals
{pendo_summary — 3 bullets}

## Ideation Summary

### Ideas explored
{Key ideas generated during brainstorming — include alternatives considered, not just the winner}

### Selected approach
{The approach the team converges on, with rationale for choosing it over alternatives}

### Risks & anti-patterns identified
{Risks surfaced: technical, UX, business}

### Adjacent opportunities
{Related features or improvements noticed during ideation — flag for future backlog}

## Research Findings
{Summary from domain/technical research, or "None — skipped"}

## Decisions for the PRD
{Explicit list of decisions and constraints the PRD must reflect. Be specific.}

## Out of scope (decided here)
{Things explicitly ruled out during brainstorming, with rationale}
```

Work section by section with the user. Do not generate the full document in one shot — ask for confirmation on the "Selected approach" and "Decisions for the PRD" before finalizing.

### Step 6 — Gate 1

Present the brainstorm summary.

> **[A] Approve — proceed to /nara-prd {slug}** | **[R] Re-run brainstorming on a different angle** | **[X] Abandon feature**

🛑 HALT until user responds.

On **[R]** → return to Step 3.
On **[X]** → confirm with user, then stop.

### Step 7 — Hand off

Output:
```
✅ Phase 1 complete — Brainstorm session done.

Artifacts:
  {feature_folder}/brainstorm.md
  {feature_folder}/research.md  ← only if research was run

▶ NEXT: /nara-prd {slug}
  The PRD will build directly on the decisions captured in brainstorm.md.
```
