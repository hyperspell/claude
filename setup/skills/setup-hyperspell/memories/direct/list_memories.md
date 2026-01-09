# List Memories

Retrieve and paginate through memories stored in Hyperspell. Use this to build admin interfaces, display user content, or manage indexed documents.

## API Reference

**Endpoint:** `GET /memories/list`

**Headers:**
- `Authorization: Bearer <API_KEY>` or `Bearer <USER_TOKEN>`
- `X-As-User: <user_id>` (only when using API key, not with user token)

**Query Parameters:**
- `limit` - Number of results per page (default: 20, max: 100)
- `offset` - Number of results to skip for pagination
- `source` - Filter by source (e.g., 'gmail', 'slack', 'custom')

## TypeScript Implementation

```typescript
import Hyperspell from 'hyperspell';

interface ListMemoriesParams {
  userId: string;
  limit?: number;
  offset?: number;
  source?: string;
}

interface Memory {
  id: string;
  title: string;
  source: string;
  created_at: string;
  metadata: Record<string, unknown>;
}

interface ListMemoriesResponse {
  memories: Memory[];
  total: number;
  limit: number;
  offset: number;
}

export async function listMemories({
  userId,
  limit = 20,
  offset = 0,
  source,
}: ListMemoriesParams): Promise<ListMemoriesResponse> {
  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await client.memories.list({
    limit,
    offset,
    source,
  });

  return response;
}
```

## Python Implementation

```python
import os
from hyperspell import Hyperspell
from typing import Optional

def list_memories(
    user_id: str,
    limit: int = 20,
    offset: int = 0,
    source: Optional[str] = None
):
    """List memories for a specific user with pagination."""
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    response = client.memories.list(
        limit=limit,
        offset=offset,
        source=source
    )

    return response
```

## Raw API (No SDK)

```typescript
interface ListOptions {
  limit?: number;
  offset?: number;
  source?: string;
}

async function listMemories(userId: string, options: ListOptions = {}) {
  const params = new URLSearchParams();
  if (options.limit) params.set('limit', String(options.limit));
  if (options.offset) params.set('offset', String(options.offset));
  if (options.source) params.set('source', options.source);

  const response = await fetch(
    `https://api.hyperspell.com/memories/list?${params}`,
    {
      headers: {
        'Authorization': `Bearer ${process.env.HYPERSPELL_API_KEY}`,
        'X-As-User': userId,
      },
    }
  );

  if (!response.ok) {
    throw new Error(`Failed to list memories: ${response.statusText}`);
  }

  return response.json();
}
```

## Building a Memories Admin UI

### React Component Example

```tsx
import { useState, useEffect } from 'react';

interface Memory {
  id: string;
  title: string;
  source: string;
  created_at: string;
}

export function MemoriesList({ userId }: { userId: string }) {
  const [memories, setMemories] = useState<Memory[]>([]);
  const [loading, setLoading] = useState(true);
  const [page, setPage] = useState(0);
  const [total, setTotal] = useState(0);
  const limit = 20;

  useEffect(() => {
    async function fetchMemories() {
      setLoading(true);
      const response = await fetch(
        `/api/memories?offset=${page * limit}&limit=${limit}`
      );
      const data = await response.json();
      setMemories(data.memories);
      setTotal(data.total);
      setLoading(false);
    }
    fetchMemories();
  }, [page]);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h2>Your Memories ({total} total)</h2>
      <ul>
        {memories.map((memory) => (
          <li key={memory.id}>
            <strong>{memory.title}</strong>
            <span>Source: {memory.source}</span>
            <span>{new Date(memory.created_at).toLocaleDateString()}</span>
          </li>
        ))}
      </ul>
      <div>
        <button onClick={() => setPage(p => Math.max(0, p - 1))} disabled={page === 0}>
          Previous
        </button>
        <span>Page {page + 1} of {Math.ceil(total / limit)}</span>
        <button onClick={() => setPage(p => p + 1)} disabled={(page + 1) * limit >= total}>
          Next
        </button>
      </div>
    </div>
  );
}
```

### API Route Handler

```typescript
// Next.js App Router: app/api/memories/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { listMemories } from '@/lib/hyperspell';
import { auth } from '@/lib/auth';

export async function GET(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const limit = parseInt(searchParams.get('limit') || '20');
  const offset = parseInt(searchParams.get('offset') || '0');
  const source = searchParams.get('source') || undefined;

  const result = await listMemories({
    userId: session.user.id,
    limit,
    offset,
    source,
  });

  return NextResponse.json(result);
}
```

## Filtering by Source

Filter memories by their source to show specific types:

```typescript
// Show only manually added memories
const customMemories = await listMemories({
  userId,
  source: 'custom',
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

- Use pagination for large memory collections to avoid performance issues
- Consider caching results for frequently accessed lists
- The `source` filter is useful for building tabbed or filtered views
- Combine with the delete endpoint to build full CRUD admin interfaces
- For search functionality, use the `/memories/query` endpoint instead
