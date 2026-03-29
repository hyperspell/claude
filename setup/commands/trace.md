---
description: Enable automatic session trace sending to Hyperspell
argument-hint: [api-key]
---

# Trace Command

Display this message:

```
Hyperspell Auto-Trace sends your Claude Code session transcripts to Hyperspell
at the end of each session. This enables:

  - Procedural memory extraction (reusable how-to knowledge from past sessions)
  - Mood capture (emotional context for relationship continuity)
```

When the user invoked this command, they may have passed the following API key
as an argument: '$1'

Follow the instructions in [SKILL.md](../skills/setup-hyperspell/tracing/SKILL.md)
to set up auto-trace. If the API key argument is present, use it during configuration.
