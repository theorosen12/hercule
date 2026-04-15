---
description: 'Execute story implementation following a context filled story spec file. Use when the user says "dev this story [story 
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-dev-story/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
