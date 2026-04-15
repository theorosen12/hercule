---
description: 'Document brownfield projects for AI context. Use when the user says "document this project" or "generate project docs"'
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-document-project/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
