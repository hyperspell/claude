# Add Memories

Add text content to Hyperspell as searchable memories. Use this for conversations, documents, notes, or any text data.

## API Reference

**Endpoint:** `POST /memories/add`

**Headers:**
- `Authorization: Bearer <API_KEY>` or `Bearer <USER_TOKEN>`
- `Content-Type: application/json`
- `X-As-User: <user_id>` (only when using API key, not with user token)

**Request Body:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `text` | string | Yes | Full text of the document |
| `resource_id` | string | No | Identifier for the resource; generates new ID if omitted |
| `collection` | string | No | Organize document for easier retrieval |
| `title` | string | No | Document title |
| `date` | string (ISO 8601) | No | Creation or update date for ranking/filtering |
| `metadata` | object | No | Custom key-value pairs (alphanumeric keys, max 64 chars) |

**Response:**
```json
{
  "source": "collections",
  "resource_id": "<string>",
  "status": "pending"
}
```

## TypeScript Implementation

Create a function to add memories. Place this in an appropriate location based on your project structure (e.g., `lib/hyperspell.ts`, `utils/memories.ts`, or alongside other API utilities).

```typescript
import Hyperspell from 'hyperspell';

interface AddMemoryParams {
  userId: string;
  text: string;
  title?: string;
  collection?: string;
  metadata?: Record<string, string | number | boolean>;
}

export async function addMemory({
  userId,
  text,
  title,
  collection,
  metadata,
}: AddMemoryParams) {
  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await client.memories.add({
    text,
    title,
    collection,
    metadata,
  });

  return response;
}
```

## Python Implementation

```python
import os
from hyperspell import Hyperspell

def add_memory(user_id: str, text: str, title: str = None, collection: str = None, metadata: dict = None):
    """Add a memory for a specific user."""
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    response = client.memories.add(
        text=text,
        title=title,
        collection=collection,
        metadata=metadata
    )

    return response
```

## Common Use Cases

### 1. Track Conversations

Add each conversation or message exchange:

```typescript
await addMemory({
  userId: currentUser.id,
  text: `User asked: "${userMessage}"\nAssistant replied: "${assistantResponse}"`,
  title: `Chat on ${new Date().toLocaleDateString()}`,
  collection: 'conversations',
  metadata: {
    type: 'conversation',
    timestamp: new Date().toISOString(),
  },
});
```

### 2. Store Document Content

```typescript
await addMemory({
  userId: currentUser.id,
  text: extractedDocumentText,
  title: documentTitle,
  collection: 'documents',
  metadata: {
    originalFilename: file.name,
    processedAt: new Date().toISOString(),
  },
});
```

### 3. Add Knowledge Base Entries

```typescript
for (const entry of knowledgeBaseEntries) {
  await addMemory({
    userId: 'system',
    text: entry.content,
    title: entry.title,
    collection: 'knowledge_base',
    metadata: {
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
- The `status` in the response will be `pending` initially - documents are processed asynchronously
