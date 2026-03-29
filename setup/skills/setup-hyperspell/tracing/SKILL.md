---
name: Enable Hyperspell Auto-Trace
description: Set up automatic session trace sending to Hyperspell for memory extraction
allowed-tools: Bash, Read, Write, Edit, AskUserQuestion
---

# Enable Hyperspell Auto-Trace

This skill configures a Claude Code `SessionEnd` hook that automatically sends your conversation transcripts to Hyperspell at the end of each session. Hyperspell extracts procedural memories and emotional context from these traces, so your agents can learn from past sessions.

## Instructions

### Step 1: Verify Prerequisites

Check that `jq` is installed:

```sh
which jq
```

If not found, display:

```
jq is required for the auto-trace hook. Install it with:
  brew install jq    (macOS)
  apt install jq     (Linux)
```

Wait for the user to install it before continuing.

### Step 2: Get the API Key

When the user invoked this skill, they may have passed an API key as an argument: '$1'

If that string is blank or not a valid API key (should start with `hs-` or `hs_`), check:

1. The current project's `.env.local` or `.env` for `HYPERSPELL_API_KEY`
2. The user's shell profile (`~/.zshrc`, `~/.bashrc`, `~/.zprofile`) for an exported `HYPERSPELL_API_KEY`

If still not found, ask the user:

```
Please paste your Hyperspell API key. Get one at https://app.hyperspell.com/api-keys
```

### Step 3: Get the User ID

Check the user's shell profile for `HYPERSPELL_USER_ID`. If not found, ask:

```
What user ID should traces be associated with? This can be your email or any identifier.
```

If the user doesn't have a preference, suggest using their system username (output of `whoami`).

### Step 4: Ensure Environment Variables Are Exported

Check the user's primary shell profile (usually `~/.zshrc` on macOS). If `HYPERSPELL_API_KEY` is not exported there, add it:

```sh
export HYPERSPELL_API_KEY="<the-api-key>"
```

Similarly for `HYPERSPELL_USER_ID` if the user provided a custom one.

**Do not duplicate** if the export already exists -- update the existing line instead.

### Step 5: Create the Trace Script

Create the directory `~/.hyperspell/` if it doesn't exist, then write the following script to `~/.hyperspell/send-trace.sh`:

```bash
#!/usr/bin/env bash
# Hyperspell Auto-Trace: Sends Claude Code session transcripts to Hyperspell
# for procedural memory and mood extraction.
#
# Invoked by a Claude Code SessionEnd hook. Reads hook JSON from stdin.

set -euo pipefail

INPUT=$(cat)

SESSION_ID=$(echo "$INPUT" | jq -r '.session_id // empty')
TRANSCRIPT_PATH=$(echo "$INPUT" | jq -r '.transcript_path // empty')

if [ -z "$TRANSCRIPT_PATH" ] || [ ! -f "$TRANSCRIPT_PATH" ]; then
  exit 0
fi

if [ -z "$SESSION_ID" ]; then
  exit 0
fi

if [ -z "${HYPERSPELL_API_KEY:-}" ]; then
  exit 0
fi

HYPERSPELL_API_URL="${HYPERSPELL_API_URL:-https://api.hyperspell.com}"
HYPERSPELL_USER_ID="${HYPERSPELL_USER_ID:-$(whoami)}"

HISTORY=$(cat "$TRANSCRIPT_PATH")

PAYLOAD=$(jq -n \
  --arg session_id "$SESSION_ID" \
  --arg history "$HISTORY" \
  '{
    session_id: $session_id,
    history: $history,
    format: "openclaw",
    extract: ["procedure", "mood"]
  }')

curl -s -X POST "${HYPERSPELL_API_URL}/trace/add" \
  -H "Authorization: Bearer ${HYPERSPELL_API_KEY}" \
  -H "X-As-User: ${HYPERSPELL_USER_ID}" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD" \
  --max-time 30 \
  > /dev/null 2>&1 || true

exit 0
```

Then make it executable:

```sh
chmod +x ~/.hyperspell/send-trace.sh
```

### Step 6: Configure the Claude Code Hook

Read the existing `~/.claude/settings.json` file (create it if it doesn't exist).

Add a `SessionEnd` hook entry that runs the trace script. **Merge** this into the existing `hooks` object -- do not overwrite other hooks.

The hook entry to add:

```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.hyperspell/send-trace.sh"
          }
        ]
      }
    ]
  }
}
```

If `hooks.SessionEnd` already has entries, append to the array. If a Hyperspell trace hook already exists (check for `send-trace.sh` in any existing command), skip adding a duplicate.

### Step 7: Confirm

Display:

```
Auto-trace is now enabled. At the end of each Claude Code session, your
conversation transcript will be sent to Hyperspell for memory extraction.

What happens:
  - Procedural memories are extracted (reusable how-to knowledge)
  - Mood/emotional context is captured (relationship continuity)
  - Traces are searchable via the Hyperspell API

To disable later, remove the SessionEnd hook from ~/.claude/settings.json
or delete ~/.hyperspell/send-trace.sh.

Restart Claude Code for the hook to take effect.
```
