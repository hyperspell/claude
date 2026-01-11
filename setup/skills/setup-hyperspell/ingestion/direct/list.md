# List Memories

Retrieve and paginate through memories stored in Hyperspell. Use this to build admin interfaces, display user content, or manage indexed documents.

## API Reference

**Endpoint:** `GET /memories/list`

**Headers:**
- `Authorization: Bearer <API_KEY>` or `Bearer <USER_TOKEN>`
- `X-As-User: <user_id>` (only when using API key, not with user token)

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `collection` | string | No | Filter documents by collection |
| `source` | enum | No | Filter by source (collections, vault, web_crawler, notion, slack, etc.) |
| `filter` | string | No | Filter by metadata using MongoDB-style operators |
| `cursor` | string | No | Pagination cursor from previous response |
| `size` | integer | No | Results per page (default: 50, max: 100) |

**Response:**
```json
{
  "items": [
    {
      "source": "collections",
      "resource_id": "<string>",
      "title": "<string>",
      "metadata": {
        "created_at": "2023-11-07T05:31:56Z",
        "indexed_at": "2023-11-07T05:31:56Z",
        "last_modified": "2023-11-07T05:31:56Z",
        "status": "pending",
        "url": "<string>"
      }
    }
  ],
  "next_cursor": "<string>"
}
```

## TypeScript Implementation

The SDK provides an async iterator for easy pagination:

```typescript
import Hyperspell from 'hyperspell';

interface ListMemoriesParams {
  userId: string;
  collection?: string;
  source?: string;
  size?: number;
}

export async function listMemories({ userId, collection, source, size }: ListMemoriesParams) {
  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const memories = [];
  for await (const memory of client.memories.list({ collection, source, size })) {
    memories.push(memory);
  }

  return memories;
}
```

## Python Implementation

```python
import os
from hyperspell import Hyperspell

def list_memories(user_id: str, collection: str = None, source: str = None, size: int = 50):
    """List memories for a specific user with pagination."""
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    memories = []
    for memory in client.memories.list(collection=collection, source=source, size=size):
        memories.append(memory)

    return memories
```

## Common Use Cases

### List all memories for a user

```typescript
const memories = await listMemories({
  userId: currentUser.id,
});
console.log(`Found ${memories.length} memories`);
```

### List memories by collection

```typescript
const conversations = await listMemories({
  userId: currentUser.id,
  collection: 'conversations',
});
```

### List memories by source

```typescript
// Show only Gmail memories
const gmailMemories = await listMemories({
  userId: currentUser.id,
  source: 'gmail',
});

// Show only Slack memories
const slackMemories = await listMemories({
  userId: currentUser.id,
  source: 'slack',
});

// Show only manually added memories
const customMemories = await listMemories({
  userId: currentUser.id,
  source: 'collections',
});
```

## Integration Notes

- Call this helper from your existing code (e.g., when displaying a user's data, in admin dashboards)
- The SDK's async iterator handles pagination automatically
- Use `collection` and `source` filters to organize different types of memories
- For search functionality, use the search API instead (see the search/ folder)
