---
description: 'Generate sprint status tracking from epics. Use when the user says "run sprint planning" or "generate sprint plan"'
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-sprint-planning/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
