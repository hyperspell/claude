# Direct Memory Management

This guide helps you add memories programmatically to Hyperspell - useful for tracking conversations, uploading documents, or ingesting data from custom sources.

## Available Operations

Display the following to the user:

```
You've chosen to add memories directly. Hyperspell provides several ways to manage memories programmatically:

1. **Add memories** - Add text content (conversations, documents, notes)
2. **Upload files** - Upload files for automatic processing
3. **List memories** - View and manage existing memories

Which operation would you like to set up?
```

Offer a multiple choice menu:
- Add memories (text content, conversations)
- Upload files
- List/manage memories
- All of the above

Based on their choice, follow the appropriate instructions:
- Add memories → [add_memories.md](add_memories.md)
- Upload files → [upload_files.md](upload_files.md)
- List memories → [list_memories.md](list_memories.md)
- All → Follow all three files in sequence

## Auth Setup for Direct Memory Operations

Before implementing any operation, ensure the auth pattern is set up correctly based on the detected flow:

**FLOW C (Backend with Direct Memory):**
- Use API key from environment variable
- Include `X-As-User` header with the user ID for each request
- No user token needed

**FLOW D (Frontend-only):**
- Need to fetch user token from external endpoint first
- Use that token for Hyperspell API calls
- See [frontend_only.md](../frameworks/frontend_only.md) for token setup

## TypeScript SDK Setup

For TypeScript/JavaScript projects, the Hyperspell SDK provides typed methods:

```typescript
import Hyperspell from 'hyperspell';

// For backend (FLOW C) - use API key with user ID
const hyperspell = new Hyperspell({
  apiKey: process.env.HYPERSPELL_API_KEY,
  userID: userId  // The ID of the user whose memories these are
});

// For frontend (FLOW D) - use user token
const hyperspell = new Hyperspell({
  apiKey: userToken  // Token fetched from your backend or auth provider
});
```

## Python SDK Setup

For Python projects:

```python
from hyperspell import Hyperspell

# For backend (FLOW C) - use API key with user ID
hyperspell = Hyperspell(
    api_key=os.environ["HYPERSPELL_API_KEY"],
    user_id=user_id  # The ID of the user whose memories these are
)

# For frontend/scripts (FLOW D) - use user token
hyperspell = Hyperspell(api_key=user_token)
```

## Common Use Cases

- **Conversation tracking**: Add each message or conversation summary as a memory
- **Document ingestion**: Upload PDFs, Word docs, or text files
- **Knowledge base**: Add FAQ entries, documentation, or reference materials
- **Activity logging**: Track user activities that should be searchable later
