---
description: 'Summarize sprint status and surface risks. Use when the user says "check sprint status" or "show sprint status"'
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-sprint-status/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
