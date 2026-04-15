---
description: 'Perform a Cynical Review and produce a findings report. Use when the user requests a critical review of something'
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-review-adversarial-general/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
