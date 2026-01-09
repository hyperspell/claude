# Add Memories

Add text content to Hyperspell as searchable memories. Use this for conversations, documents, notes, or any text data.

## API Reference

**Endpoint:** `POST /memories/add`

**Headers:**
- `Authorization: Bearer <API_KEY>` or `Bearer <USER_TOKEN>`
- `Content-Type: application/json`
- `X-As-User: <user_id>` (only when using API key, not with user token)

**Request Body:**
```json
{
  "content": "The text content to add as a memory",
  "title": "Optional title for the memory",
  "source": "custom",
  "metadata": {
    "key": "value"
  }
}
```

## TypeScript Implementation

Create a function to add memories. Place this in an appropriate location based on your project structure (e.g., `lib/hyperspell.ts`, `utils/memories.ts`, or alongside other API utilities).

```typescript
import Hyperspell from 'hyperspell';

const hyperspell = new Hyperspell({
  apiKey: process.env.HYPERSPELL_API_KEY!,
});

interface AddMemoryParams {
  userId: string;
  content: string;
  title?: string;
  metadata?: Record<string, unknown>;
}

export async function addMemory({ userId, content, title, metadata }: AddMemoryParams) {
  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await client.memories.add({
    content,
    title,
    source: 'custom',
    metadata,
  });

  return response;
}
```

## Python Implementation

```python
import os
from hyperspell import Hyperspell

def add_memory(user_id: str, content: str, title: str = None, metadata: dict = None):
    """Add a memory for a specific user."""
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    response = client.memories.add(
        content=content,
        title=title,
        source="custom",
        metadata=metadata or {}
    )

    return response
```

## Raw API (No SDK)

If not using the SDK, make direct HTTP requests:

```typescript
async function addMemory(userId: string, content: string, title?: string) {
  const response = await fetch('https://api.hyperspell.com/memories/add', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.HYPERSPELL_API_KEY}`,
      'Content-Type': 'application/json',
      'X-As-User': userId,
    },
    body: JSON.stringify({
      content,
      title,
      source: 'custom',
    }),
  });

  if (!response.ok) {
    throw new Error(`Failed to add memory: ${response.statusText}`);
  }

  return response.json();
}
```

## Common Use Cases

### 1. Track Conversations

Add each conversation or message exchange:

```typescript
// After a chat interaction
await addMemory({
  userId: currentUser.id,
  content: `User asked: "${userMessage}"\nAssistant replied: "${assistantResponse}"`,
  title: `Chat on ${new Date().toLocaleDateString()}`,
  metadata: {
    type: 'conversation',
    timestamp: new Date().toISOString(),
  },
});
```

### 2. Store Document Content

```typescript
// After processing a document
await addMemory({
  userId: currentUser.id,
  content: extractedDocumentText,
  title: documentTitle,
  metadata: {
    type: 'document',
    originalFilename: file.name,
    processedAt: new Date().toISOString(),
  },
});
```

### 3. Add Knowledge Base Entries

```typescript
// Programmatically add FAQ or reference content
for (const entry of knowledgeBaseEntries) {
  await addMemory({
    userId: 'system', // or a specific user
    content: entry.content,
    title: entry.title,
    metadata: {
      type: 'knowledge_base',
      category: entry.category,
    },
  });
}
```

## Integration Notes

- Place the helper function in a location accessible from where you need to add memories
- For server-side frameworks (Express, FastAPI, Next.js API routes), call this from your API handlers
- For conversation tracking, consider batching or debouncing to avoid too many API calls
- Use meaningful titles and metadata to improve search relevance
