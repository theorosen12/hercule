---
description: 'Analyzes current state and user query to answer BMad questions or recommend the next workflow or agent. Use when user s
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-help/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
