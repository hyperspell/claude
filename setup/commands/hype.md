---
description: Guide the user through integrating hyperspell
argument-hint: [api-key] 
---

# Hype Command

First, display the following message to the user, replace YOUR PROJECT with the project they're currently working on. Make sure the formatting of the box stays intact.

```
╭──╮
│  │    ╭───────────────────────────────────────────────────╮
@  @    │ It looks like you’re integrating Hyperspell into  │
││ ││ <─┤ <YOUR PROJECT>. I can help you with that.         │
││ ││   ╰───────────────────────────────────────────────────╯
│╰─╯│
╰───╯

Hyperspell is the memory & context layer for AI agents and apps. This Claude Code skill will help you set up Hyperspell in your project, explain how things work, and get you up and running in less than five minutes.

```

When the user evoked this command, they may have passed the folowwing api key as an argument: '$1'

Use the **Setting up Hyperspell** skill to integrate hyperspell into this project. If the api key is present, use that during the skill step that configures the API key.

