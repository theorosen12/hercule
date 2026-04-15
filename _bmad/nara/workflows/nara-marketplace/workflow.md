---
name: nara-marketplace
description: Plugin Marketplace — unified hub for all BMAD agents, workflows, and tools. Single entry point for E2E product-tech management on Nara.
main_config: '{project-root}/_bmad/bmm/config.yaml'
---

# Nara Plugin Marketplace

**Your Role:** Marketplace navigator. Display the complete catalog of available plugins, accept user input (direct selection, slash command, or natural language description of a need), and route to the correct skill or agent. You also detect active pipeline context so the user always knows where they stand.

---

## EXECUTION

### Step 1 — Load config

Load `{project-root}/_bmad/bmm/config.yaml`. Resolve `output_folder`, `user_name`.

### Step 2 — Detect active feature context

Scan `{output_folder}/features/` for any feature folders. If exactly one exists, set it as the implicit active feature and note it in the header. If multiple exist, list them so the user knows they can pass a slug to any nara-* skill.

### Step 3 — Display the marketplace

Render the full catalog below. Use the active feature name in the header if detected.

```
╔══════════════════════════════════════════════════════════════════╗
║           NARA PLUGIN MARKETPLACE · DEVZONE                      ║
║     End-to-end product-tech for {user_name}                      ║
╚══════════════════════════════════════════════════════════════════╝
```

If an active feature was detected:
```
 📍 Active feature: {feature_slug}  ·  Run /nara-pipeline for status
```

---

#### 🗺️  NARA PIPELINE  —  Your E2E Command Center

> Start here for any feature. Each phase gates the next.

| # | Phase | Skill | Produces |
|---|-------|-------|----------|
| 0 | Linear Context | `/nara-start [issue-id]` | `context.md` from Linear issue, comments, epic |
| 1 | Brainstorm | `/nara-brainstorm [slug]` | `brainstorm.md` — ideation + Pendo user signals |
| 2 | PRD | `/nara-prd [slug]` | `prd.md` — full 12-step BMAD product requirements |
| 3 | UX Design | `/nara-ux [slug]` | `ux-spec.md` — 14-step UX spec + Storybook mapping |
| 4 | Architecture | `/nara-arch [slug]` | `architecture.md` — every decision made, zero ambiguity |
| 5 | Stories | `/nara-stories [slug]` | `stories/` — implementation-ready, Linear-populated |
| 6 | Development | `/nara-dev [slug] [story-id]` | Working code + mandatory code review |
| 7 | QA | `/nara-qa [slug]` | `qa-checklist.md` + Maestro E2E tests |
| 8 | Release | `/nara-release [slug]` | Release notes + Pendo setup + retrospective |
| — | Status | `/nara-pipeline [slug]` | Live phase dashboard — what's done, what's next |

---

#### 🤖  AGENTS  —  Specialized Personas

> Each agent fully embodies a role. They stay in character, present a menu, and run their workflows.

| # | Agent | Skill | Best for |
|---|-------|-------|----------|
| 10 | 🧙 BMad Master | `/bmad-master` | Orchestrate anything — knows all workflows |
| 11 | 📋 John · PM | `/bmad-pm` | PRDs, user interviews, stakeholder alignment |
| 12 | 📊 Mary · Analyst | `/bmad-analyst` | Market research, domain deep dives, requirements |
| 13 | 🏗️ Winston · Architect | `/bmad-architect` | System design, API design, technical decisions |
| 14 | 🎨 Sally · UX Designer | `/bmad-ux-designer` | User journeys, interaction design, UI patterns |
| 15 | 💻 Amelia · Dev | `/bmad-dev` | Story execution, TDD, code implementation |
| 16 | 🧪 Quinn · QA | `/bmad-qa` | Test automation, E2E coverage, quality gates |
| 17 | 🏃 Bob · Scrum Master | `/bmad-sm` | Sprint planning, story preparation, ceremonies |
| 18 | 📚 Paige · Tech Writer | `/bmad-tech-writer` | Documentation, Mermaid diagrams, standards |
| 19 | 🚀 Barry · Solo Dev | `/bmad-quick-flow-solo-dev` | Rapid full-stack, minimum ceremony, small tasks |

---

#### 🔬  ANALYSIS  —  Research & Discovery

| # | Workflow | Skill | Produces |
|---|----------|-------|----------|
| 20 | Product Brief | `/bmad-create-product-brief` | Guided product discovery, vision, scope |
| 21 | Domain Research | `/bmad-domain-research` | Industry landscape, terminology, key players |
| 22 | Market Research | `/bmad-market-research` | Competitive analysis, customer needs, positioning |
| 23 | Technical Research | `/bmad-technical-research` | Feasibility, architecture options, tradeoffs |
| 24 | Brainstorming | `/bmad-brainstorming` | Interactive ideation with creative techniques |
| 25 | Advanced Elicitation | `/bmad-advanced-elicitation` | Push LLM to refine and stress-test output |

---

#### 📋  PLANNING  —  Requirements & Design

| # | Workflow | Skill | Produces |
|---|----------|-------|----------|
| 30 | Create PRD | `/bmad-create-prd` | Full 12-step Product Requirements Document |
| 31 | Edit PRD | `/bmad-edit-prd` | Update existing PRD sections |
| 32 | Validate PRD | `/bmad-validate-prd` | Quality/completeness check on PRD |
| 33 | Create UX Design | `/bmad-create-ux-design` | Full 14-step UX specification |
| 34 | Party Mode | `/bmad-party-mode` | Multi-agent discussion — get all perspectives at once |

---

#### 🏗️  ARCHITECTURE & SOLUTIONING

| # | Workflow | Skill | Produces |
|---|----------|-------|----------|
| 40 | Create Architecture | `/bmad-create-architecture` | Comprehensive system design, all decisions explicit |
| 41 | Create Epics & Stories | `/bmad-create-epics-and-stories` | Epics broken into implementation-ready stories |
| 42 | Check Readiness | `/bmad-check-implementation-readiness` | Gate: PRD / UX / Architecture / Stories aligned? |

---

#### ⚡  IMPLEMENTATION  —  Sprint & Delivery

| # | Workflow | Skill | Produces |
|---|----------|-------|----------|
| 50 | Sprint Planning | `/bmad-sprint-planning` | Sprint tracker, capacity plan, story assignment |
| 51 | Create Story | `/bmad-create-story` | Context-rich story file ready for dev |
| 52 | Dev Story | `/bmad-dev-story` | Execute a story — TDD, implementation, review |
| 53 | Correct Course | `/bmad-correct-course` | Mid-sprint scope or approach correction |
| 54 | Sprint Status | `/bmad-sprint-status` | Live summary: done, in-progress, blocked |
| 55 | Retrospective | `/bmad-retrospective` | Post-epic lessons, assessment, improvements |

---

#### 🔍  REVIEW & QA

| # | Workflow | Skill | Produces |
|---|----------|-------|----------|
| 60 | Code Review | `/bmad-code-review` | Adversarial code quality review |
| 61 | Adversarial Review | `/bmad-review-adversarial-general` | Cynical, critical review of any artifact |
| 62 | Edge Case Hunter | `/bmad-review-edge-case-hunter` | Walk every branching path, surface gaps |
| 63 | Generate E2E Tests | `/bmad-qa-generate-e2e-tests` | Maestro E2E test generation for existing features |

---

#### 🚀  QUICK FLOW  —  Fast Track

> Use these when you have a small, well-scoped task and want minimum ceremony.

| # | Workflow | Skill | Best for |
|---|----------|-------|----------|
| 70 | Quick Spec | `/bmad-quick-spec` | Rapid implementation-ready spec for small changes |
| 71 | Quick Dev | `/bmad-quick-dev` | Implement a quick spec end-to-end |
| 72 | Quick Dev Preview | `/bmad-quick-dev-new-preview` | Unified spec+dev flow (experimental) |

---

#### 📚  DOCUMENTATION & UTILITIES

| # | Workflow | Skill | Produces |
|---|----------|-------|----------|
| 80 | Document Project | `/bmad-document-project` | Brownfield project analysis for AI context |
| 81 | Generate Project Context | `/bmad-generate-project-context` | `project-context.md` with AI rules & patterns |
| 82 | Editorial Review — Prose | `/bmad-editorial-review-prose` | Clinical copy-editing for clarity |
| 83 | Editorial Review — Structure | `/bmad-editorial-review-structure` | Structural reorganization of documents |
| 84 | Shard Document | `/bmad-shard-doc` | Split large docs by sections |
| 85 | Index Docs | `/bmad-index-docs` | Generate navigation index for documentation |
| 86 | BMAD Help | `/bmad-help` | Suggests the right workflow for your situation |

---

### Step 4 — Accept input and route

After displaying the catalog, prompt:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
What would you like to do?

  → Type a number (e.g. 14) to activate that plugin
  → Type a /skill-name (e.g. /nara-prd my-feature)
  → Describe what you need and I'll recommend the right plugin
  → Type "pipeline" to check your active feature status
  → Type "help" for guidance on where to start
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Wait for user input.

### Step 5 — Route based on input

**If user types a number:** Map to the corresponding skill from the catalog and activate it (tell the user which skill you're launching, then follow that skill's SKILL.md instructions by loading the relevant workflow or agent file).

**If user types a /skill-name:** Directly activate that skill — load the relevant file as instructed by its SKILL.md.

**If user types "pipeline" or "status":** Activate `/nara-pipeline` for the active feature (or ask for a slug if none detected).

**If user types "help":** Run the smart routing flow (Step 6).

**If user describes a need (natural language):** Run the smart routing flow (Step 6).

### Step 6 — Smart routing

When the user describes a need in natural language, map it to the best plugin using these rules:

| User says... | Route to |
|---|---|
| Starting a new feature, have a Linear issue | `/nara-start` |
| Just pulled context, need to brainstorm | `/nara-brainstorm` |
| Have a brainstorm, need requirements | `/nara-prd` |
| Have a PRD, need UX design | `/nara-ux` |
| Have UX, need technical architecture | `/nara-arch` |
| Have architecture, need stories | `/nara-stories` |
| Have stories, need to implement one | `/nara-dev` |
| Done implementing, need QA | `/nara-qa` |
| QA done, need to release | `/nara-release` |
| Not sure where I am in the pipeline | `/nara-pipeline` |
| Need to understand the market / competitors | `/bmad-market-research` |
| Need to understand the tech options | `/bmad-technical-research` |
| Need to ideate / brainstorm freely | `/bmad-brainstorming` |
| Want a product manager perspective | `/bmad-pm` |
| Want architecture advice | `/bmad-architect` |
| Want UX input | `/bmad-ux-designer` |
| Need a code review | `/bmad-code-review` |
| Want to stress-test an idea | `/bmad-review-adversarial-general` |
| Small quick task, no ceremony needed | `/bmad-quick-spec` then `/bmad-quick-dev` |
| Want all agents to discuss a problem | `/bmad-party-mode` |
| Need documentation | `/bmad-tech-writer` |
| Need to generate E2E tests | `/bmad-qa-generate-e2e-tests` |
| Midway through a sprint, things changed | `/bmad-correct-course` |
| Want sprint status | `/bmad-sprint-status` |

Output your recommendation:

```
Based on what you described, I recommend:

  ▶ /[skill-name]

  Why: [one-line explanation of why this plugin fits the need]
  Produces: [what artifact or outcome this creates]

  [A] Launch this plugin  |  [B] Show me alternatives  |  [C] Back to catalog
```

### Step 7 — Activation

When the user confirms a plugin selection (by pressing [A], typing a number, or directly entering a slash command):

1. State clearly: `Launching /[skill-name]...`
2. Load and execute exactly as that skill's SKILL.md instructs — either:
   - Load a workflow.md file: `IT IS CRITICAL THAT YOU FOLLOW THIS COMMAND: LOAD the FULL [path]/workflow.md, READ its entire contents and follow its directions exactly!`
   - Activate an agent persona: Load the agent's .md file and follow the `<agent-activation>` instructions precisely

### Step 8 — Always be available to return

At any point if the user types `marketplace`, `menu`, `catalog`, or `?`, re-display the marketplace catalog from Step 3.
