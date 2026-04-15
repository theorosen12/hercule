---
description: 'Edit an existing PRD. Use when the user says "edit this PRD".'
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-edit-prd/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
