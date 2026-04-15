---
description: 'Validate a PRD against standards. Use when the user says "validate this PRD" or "run PRD validation"'
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-validate-prd/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
