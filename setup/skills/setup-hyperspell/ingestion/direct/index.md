# Direct Memory Management

This guide helps you add memories programmatically to Hyperspell - useful for tracking conversations, uploading documents, or ingesting data from custom sources.

## Instructions

Create helper functions for all three memory operations:

1. **Add memories** - Follow [add.md](add.md)
2. **Upload files** - Follow [upload.md](upload.md)
3. **List memories** - Follow [list.md](list.md)

Place these helpers in an appropriate location (e.g., `lib/hyperspell.ts`, `utils/memories.ts`, or similar).

**Important:** These are **internal helper functions** - do NOT create new API endpoints. After creating the helpers:

1. Look for obvious places in the codebase to integrate them:
   - After chat/conversation responses → call `addMemory()`
   - In existing file upload handlers → call `uploadFile()`
   - In admin/dashboard pages → call `listMemories()`

2. If there are no obvious integration points, note in your wrap-up message where the helpers are located and when the user should call them.

## Auth Setup

**Backend apps (most common):**
- Use API key from environment variable
- Pass the user ID to the SDK
- No user token needed

**Frontend-only apps:**
- Need to fetch user token from a backend endpoint first
- Your backend calls Hyperspell's `/auth/user_token` endpoint to generate tokens
- See the token endpoint examples in [oauth.md](../oauth.md)

## SDK Setup

### TypeScript

```typescript
import Hyperspell from 'hyperspell';

// For backend - use API key with user ID
const hyperspell = new Hyperspell({
  apiKey: process.env.HYPERSPELL_API_KEY!,
  userID: userId  // The ID of the user whose memories these are
});
```

### Python

```python
from hyperspell import Hyperspell
import os

# For backend - use API key with user ID
hyperspell = Hyperspell(
    api_key=os.environ["HYPERSPELL_API_KEY"],
    user_id=user_id  # The ID of the user whose memories these are
)
```
