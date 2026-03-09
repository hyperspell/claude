# Claude Skills for Hyperspell

```
╭──╮
│  │    ╭───────────────────────────────────────────────────╮
@  @    │ It looks like you’re integrating Hyperspell into  │
││ ││ <─┤ your project. I can help you with that.           │
││ ││   ╰───────────────────────────────────────────────────╯
│╰─╯│
╰───╯
```

## Installation

1. Add the Hyperspell skills marketplace to claude with
    ```
    /plugin marketplace add hyperspell/claude
    ```
2. Install the setup plugin
    ```
    /plugin install setup@hyperspell
    ```
3. Run the `/setup:hype [HYPERSPELL-API-KEY]` command to get started. 

## Troubleshooting

### "Unknown skill" after installing

There's a known bug in Claude Code v2.x where plugins get added to `installed_plugins.json` but don't get auto-enabled in `settings.json`, so skills silently fail to load.

**Fix:** Manually add the plugin to `~/.claude/settings.json`:

```json
"enabledPlugins": {
    "setup@hyperspell": true
}
```

Then re-run `/setup:hype [YOUR-API-KEY]`.

See [claude-code#17832](https://github.com/anthropics/claude-code/issues/17832) for details.
## Development

Clone this repo and add the marketplace to Claude with

```
/plugin marketplace add ~/path-to-repo
```

Then install the plugin with 

```
/plugin install setup@hyperspell
```

Every time you make changes you need to re-install your plugin:

```
/plugin uninstall setup@hyperspell
/plugin install setup@hyperspell
```
