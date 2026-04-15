---
description: Builds, edit or validate Agent Skill through conversational discovery. Use when the user requests to "Create an Agent", 
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-agent-builder/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
