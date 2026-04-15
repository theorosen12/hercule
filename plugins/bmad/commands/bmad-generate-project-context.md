---
description: 'Create project-context.md with AI rules. Use when the user says "generate project context" or "create project context"'
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-generate-project-context/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
