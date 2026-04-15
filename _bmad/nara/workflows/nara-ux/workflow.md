---
name: nara-ux
description: Phase 2 — Full UX design using bmad-create-ux-design (14 steps) followed by the Nara Storybook layer (component inventory + prototype stories).
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara UX Spec — Phase 2

**Your Role:** UX designer working within the Nara design system. This phase has two layers:
1. **BMAD UX Design** — full 14-step collaborative UX specification (emotional response, design directions, user journeys, component strategy)
2. **Nara Storybook layer** — map every screen to existing Storybook components, generate prototype stories for new components

Every screen must use existing Storybook components first. New components are created only when nothing existing fits.

## EXECUTION

### Step 1 — Load config & verify prerequisite

Load config. Resolve `{feature_folder}` from argument or ask user.

Check that `{feature_folder}/prd.md` exists. If not → stop and tell user to run `/nara-prd` first.

Read `{feature_folder}/prd.md` fully.

If `{feature_folder}/brainstorm.md` exists → read it for design intent and selected approach context.

### Step 2 — Full BMAD UX Design workflow

Load and follow `{project-root}/_bmad/bmm/workflows/2-plan-workflows/create-ux-design/workflow.md`.

When loading this workflow, inject the following Nara-specific context:
- **Output file:** `{feature_folder}/ux-spec.md`
- **Platform:** React Native (Expo) mobile app — no web, no desktop
- **Design system:** Nara has an existing Storybook component library at `/mobile/src/`. Components must be inventoried and reused (see Step 3 below)
- **Design constraints:** All states must be defined (Loading, Empty, Error, Populated). Offline states are required where data is involved. Push notification flows must be included if the feature triggers notifications.
- **Tone & style:** Nara's aesthetic is warm, memory-preserving, emotionally resonant. The UX should feel like a personal travel journal, not a utility app.

Follow all 14 steps of the BMAD UX workflow without skipping. This includes:
- Emotional response design
- Multiple design directions exploration
- User journey mapping
- Component strategy
- UX patterns
- Accessibility considerations

Do not proceed to Step 3 until the user has approved the BMAD UX specification.

### Step 3 — Nara Storybook component inventory

Now apply the Nara-specific layer on top of the approved UX spec.

Using the Storybook MCP, query all stories from `/mobile/src/`. Group by domain:

For each story, extract:
- Component name
- File path
- Available props / controls
- Visual states already covered

### Step 4 — Map UX spec to Storybook components

For each screen defined in the UX spec:
- Identify which existing Storybook components can be reused
- For each component: note the exact story path and props to use
- Flag where no existing component fits → mark as "new component required"

Update `{feature_folder}/ux-spec.md` to add a **Component Mapping** section:

```markdown
## Component Mapping

### Reused from Design System
| Screen | Component | Story path | Props / Usage |
|--------|-----------|-----------|---------------|
| ...    | ...       | ...       | ...           |

### New Components Required
| Screen | Component name | Props API | Description |
|--------|---------------|-----------|-------------|
| ...    | ...           | ...       | ...         |
```

### Step 5 — Generate Storybook prototype stories

For each **new component** identified in Step 4, write a `.stories.tsx` file following the existing pattern in `/mobile/src/`.

Rules:
- Place in correct domain path: `/mobile/src/domains/{domain}/view/components/{ComponentName}/{ComponentName}.stories.tsx`
- Use mock data — never import real services
- Cover at minimum: `Default`, `Loading`, `Empty`, `Error` stories
- Use `@storybook/react-native` patterns matching existing stories

For each **modified existing component**, add new stories to the existing `.stories.tsx` file if new states are required.

Save all story files.

### Step 6 — Gate 2

Present:
- Summary of the UX spec (screens, flows, key design decisions)
- List of all generated/modified story files
- How to view them: `cd mobile && npx expo start --storybook`

> Please review all generated stories in Storybook before approving.
> **[A] Approve UX — stories look correct** | **[R] Request changes** (describe what to fix)

🛑 HALT until [A]. On [R] → apply changes → re-present.

### Step 7 — Update Linear

Using `mcp__claude_ai_Linear__create_document`, attach `ux-spec.md` to the feature's Linear project.

### Step 8 — Hand off

Update `ux-spec.md` frontmatter: `stepsCompleted: [ux-design, ux-spec, component-mapping, stories-generated, ux-approved]`

Output:
```
✅ Phase 2 complete — UX design approved, component mapping done, {N} Storybook stories generated.

Artifacts:
  {feature_folder}/ux-spec.md
  {N} story files in /mobile/src/

▶ NEXT: /nara-arch {slug}
```
