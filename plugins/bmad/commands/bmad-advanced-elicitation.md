---
description: 'Push the LLM to reconsider, refine, and improve its recent output. Use when user asks for deeper critique or mentions a
argument-hint: [optional context or input]
allowed-tools: Read
---

Read and follow the instructions in @${CLAUDE_PLUGIN_ROOT}/skills/bmad-advanced-elicitation/SKILL.md.

Task: $ARGUMENTS

If no input is provided, ask the user what they need before starting.
