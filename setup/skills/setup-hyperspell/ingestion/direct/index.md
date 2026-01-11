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
- Add memories → [add.md](add.md)
- Upload files → [upload.md](upload.md)
- List memories → [list.md](list.md)
- All → Follow all three files in sequence

## Auth Setup for Direct Memory Operations

Before implementing any operation, ensure the auth pattern is set up correctly:

**Backend apps (most common):**
- Use API key from environment variable
- Pass the user ID to the SDK
- No user token needed

**Frontend-only apps:**
- Need to fetch user token from a backend endpoint first
- Your backend calls Hyperspell's `/auth/user_token` endpoint to generate tokens
- Use that token for Hyperspell API calls from the frontend
- See the token endpoint examples in [oauth.md](../oauth.md) for how to create the backend endpoint

## TypeScript SDK Setup

For TypeScript/JavaScript projects, the Hyperspell SDK provides typed methods:

```typescript
import Hyperspell from 'hyperspell';

// For backend - use API key with user ID
const hyperspell = new Hyperspell({
  apiKey: process.env.HYPERSPELL_API_KEY!,
  userID: userId  // The ID of the user whose memories these are
});

// For frontend - use user token
const hyperspell = new Hyperspell({
  apiKey: userToken  // Token fetched from your backend or auth provider
});
```

## Python SDK Setup

For Python projects:

```python
from hyperspell import Hyperspell
import os

# For backend - use API key with user ID
hyperspell = Hyperspell(
    api_key=os.environ["HYPERSPELL_API_KEY"],
    user_id=user_id  # The ID of the user whose memories these are
)

# For frontend/scripts - use user token
hyperspell = Hyperspell(api_key=user_token)
```

## Common Use Cases

- **Conversation tracking**: Add each message or conversation summary as a memory
- **Document ingestion**: Upload PDFs, Word docs, or text files
- **Knowledge base**: Add FAQ entries, documentation, or reference materials
- **Activity logging**: Track user activities that should be searchable later
