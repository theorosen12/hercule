---
description: 'Creates a dedicated story file with all the context the agent will need to implement it later. Use when the user says "
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-create-story/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
