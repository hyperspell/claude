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

## Building a Memories Admin UI

### React Component Example

```tsx
import { useState, useEffect } from 'react';

interface Memory {
  resource_id: string;
  title: string;
  source: string;
  metadata: {
    created_at: string;
    status: string;
  };
}

export function MemoriesList({ userId }: { userId: string }) {
  const [memories, setMemories] = useState<Memory[]>([]);
  const [loading, setLoading] = useState(true);
  const [cursor, setCursor] = useState<string | null>(null);
  const [hasMore, setHasMore] = useState(false);

  async function fetchMemories(nextCursor?: string) {
    setLoading(true);
    const params = new URLSearchParams({ size: '20' });
    if (nextCursor) params.set('cursor', nextCursor);

    const response = await fetch(`/api/memories?${params}`);
    const data = await response.json();

    if (nextCursor) {
      setMemories(prev => [...prev, ...data.items]);
    } else {
      setMemories(data.items);
    }
    setCursor(data.next_cursor);
    setHasMore(!!data.next_cursor);
    setLoading(false);
  }

  useEffect(() => {
    fetchMemories();
  }, []);

  if (loading && memories.length === 0) return <div>Loading...</div>;

  return (
    <div>
      <h2>Your Memories</h2>
      <ul>
        {memories.map((memory) => (
          <li key={memory.resource_id}>
            <strong>{memory.title || 'Untitled'}</strong>
            <span>Source: {memory.source}</span>
            <span>{new Date(memory.metadata.created_at).toLocaleDateString()}</span>
          </li>
        ))}
      </ul>
      {hasMore && (
        <button onClick={() => fetchMemories(cursor!)} disabled={loading}>
          {loading ? 'Loading...' : 'Load More'}
        </button>
      )}
    </div>
  );
}
```

### API Route Handler

```typescript
// Next.js App Router: app/api/memories/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Hyperspell from 'hyperspell';

export async function GET(request: NextRequest) {
  const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules

  const { searchParams } = new URL(request.url);
  const collection = searchParams.get('collection') || undefined;
  const cursor = searchParams.get('cursor') || undefined;
  const size = parseInt(searchParams.get('size') || '20');

  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  // For paginated results, use the raw list with cursor
  const response = await fetch(
    `https://api.hyperspell.com/memories/list?${new URLSearchParams({
      ...(collection && { collection }),
      ...(cursor && { cursor }),
      size: String(size),
    })}`,
    {
      headers: {
        'Authorization': `Bearer ${process.env.HYPERSPELL_API_KEY}`,
        'X-As-User': userId,
      },
    }
  );

  const data = await response.json();
  return NextResponse.json(data);
}
```

## Filtering by Collection

Filter memories by their collection to show specific types:

```typescript
// Show only conversation memories
const conversations = await listMemories({
  userId,
  collection: 'conversations',
});

// Show only document uploads
const documents = await listMemories({
  userId,
  collection: 'documents',
});
```

## Filtering by Source

Filter by where memories originated:

```typescript
// Show only manually added memories
const customMemories = await listMemories({
  userId,
  source: 'collections',
});

// Show only Gmail memories
const gmailMemories = await listMemories({
  userId,
  source: 'gmail',
});

// Show only Slack memories
const slackMemories = await listMemories({
  userId,
  source: 'slack',
});
```

## Integration Notes

- Use cursor-based pagination for large memory collections
- The SDK's async iterator handles pagination automatically
- The `source` filter is useful for building tabbed or filtered views
- Consider caching results for frequently accessed lists
- For search functionality, use the `/memories/query` endpoint instead
