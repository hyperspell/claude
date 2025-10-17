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
3. Run the `/hype` command to get started. If you already have a Hyperspell API key, you can provide it with `/hype YOUR-API-KEY`, otherwise Claude will help you create one.

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
