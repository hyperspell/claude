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
