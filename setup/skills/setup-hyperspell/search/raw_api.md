# Raw API Integration (No SDK)

This guide covers integrating Hyperspell when you're not using a specific AI SDK, or when you need direct API access.

## Understanding the `answer` Parameter

The memory search API has an `answer` parameter that changes what you get back:

- **`answer: true`** - Hyperspell's AI generates an answer from the memories, plus returns source documents. Use this when you want Hyperspell to handle the AI response.

- **`answer: false`** - Returns only the relevant source documents/highlights. Use this when you want raw context to inject into your own AI call or use directly.

## Direct Search Function

Create a reusable search function:

### TypeScript

```typescript
// lib/hyperspell.ts
import Hyperspell from 'hyperspell';

export async function searchMemories(
  userId: string,
  query: string,
  options: { answer?: boolean; sources?: string[] } = {}
) {
  const { answer = true, sources } = options;

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await hyperspell.memories.search({
    query,
    answer,
    sources,
  });

  return response;
}
```

### Python

```python
# hyperspell_utils.py
import os
from hyperspell import Hyperspell
from typing import Optional, List

def search_memories(
    user_id: str,
    query: str,
    answer: bool = True,
    sources: Optional[List[str]] = None
):
    """Search user memories."""
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    response = client.memories.search(
        query=query,
        answer=answer,
        sources=sources
    )

    return response
```

## Without the SDK (Raw HTTP)

If you prefer not to use the Hyperspell SDK:

### TypeScript with Fetch

```typescript
// lib/hyperspell.ts

interface SearchParams {
  query: string;
  answer?: boolean;
  sources?: string[];
}

interface SearchResponse {
  answer: string;
  highlights: Array<{
    id: string;
    text: string;
    source: string;
  }>;
}

export async function searchMemories(
  userId: string,
  params: SearchParams
): Promise<SearchResponse> {
  const response = await fetch('https://api.hyperspell.com/memories/query', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.HYPERSPELL_API_KEY}`,
      'Content-Type': 'application/json',
      'X-As-User': userId,
    },
    body: JSON.stringify({
      query: params.query,
      answer: params.answer ?? true,
      sources: params.sources,
    }),
  });

  if (!response.ok) {
    const error = await response.text();
    throw new Error(`Hyperspell search failed: ${error}`);
  }

  return response.json();
}
```

### Python with Requests

```python
# hyperspell_utils.py
import os
import requests
from typing import Optional, List, Dict, Any

def search_memories(
    user_id: str,
    query: str,
    answer: bool = True,
    sources: Optional[List[str]] = None
) -> Dict[str, Any]:
    """Search user memories using raw HTTP."""
    response = requests.post(
        'https://api.hyperspell.com/memories/query',
        headers={
            'Authorization': f'Bearer {os.environ["HYPERSPELL_API_KEY"]}',
            'Content-Type': 'application/json',
            'X-As-User': user_id,
        },
        json={
            'query': query,
            'answer': answer,
            'sources': sources,
        }
    )

    response.raise_for_status()
    return response.json()
```

## Integration Patterns

### Pattern A: Direct Answer from Hyperspell (`answer: true`)

Let Hyperspell's AI answer questions directly from memories. Best for simple Q&A without other tools.

```typescript
// Simple Q&A endpoint - Hyperspell handles the AI response
app.post('/ask', async (req, res) => {
  const { question } = req.body;
  const userId = req.user.id;

  // Hyperspell returns an AI-generated answer
  const result = await searchMemories(userId, question, { answer: true });

  res.json({ answer: result.answer, sources: result.highlights });
});
```

### Pattern B: Context Retrieval for Custom AI (`answer: false`)

Get raw context from memories to use in your own AI call. Best for RAG pipelines or when you need control over the prompt.

```typescript
// RAG pattern - get context, then use your own LLM
app.post('/ask', async (req, res) => {
  const { question } = req.body;
  const userId = req.user.id;

  // Get only the relevant source documents
  const result = await searchMemories(userId, question, { answer: false });

  // Build context from highlights
  const context = result.highlights
    .map(h => h.text)
    .join('\n\n');

  // Use context in your own AI call
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: `Answer based on this context from the user's memories:\n\n${context}`
      },
      { role: 'user', content: question }
    ],
  });

  res.json({ answer: response.choices[0].message.content });
});
```

### Pattern C: Simple Conversational Agent (No Tools)

For a chatbot that uses Hyperspell for all memory-related questions:

```typescript
class SimpleMemoryChatbot {
  private userId: string;

  constructor(userId: string) {
    this.userId = userId;
  }

  async chat(userMessage: string): Promise<string> {
    // Let Hyperspell answer questions about the user's data
    const result = await searchMemories(this.userId, userMessage, { answer: true });

    // If Hyperspell found relevant info, return its answer
    if (result.answer && result.highlights.length > 0) {
      return result.answer;
    }

    // Fallback for questions not related to memories
    return "I couldn't find relevant information in your connected accounts.";
  }
}
```

### Pattern D: Hybrid - Selective Memory Search

Search memories only when relevant, use your own AI for everything else:

```typescript
async function hybridChat(userId: string, message: string): Promise<string> {
  // Check if this seems like a question about the user's data
  const memoryKeywords = ['email', 'message', 'document', 'meeting', 'said', 'wrote'];
  const shouldSearchMemories = memoryKeywords.some(kw => message.toLowerCase().includes(kw));

  let context = '';
  if (shouldSearchMemories) {
    const result = await searchMemories(userId, message, { answer: false });
    context = result.highlights.map(h => h.text).join('\n\n');
  }

  // Your AI call with optional memory context
  const systemPrompt = context
    ? `You have access to the user's memories:\n\n${context}\n\nAnswer their question.`
    : 'You are a helpful assistant.';

  const response = await yourLLM.generate(systemPrompt, message);
  return response;
}
```

## API Reference

### Search Endpoint

**POST** `https://api.hyperspell.com/memories/query`

**Headers:**
- `Authorization: Bearer <API_KEY>`
- `Content-Type: application/json`
- `X-As-User: <user_id>` (when using API key)

**Request Body:**
```json
{
  "query": "What did John say about the project deadline?",
  "answer": true,
  "sources": ["gmail", "slack"]
}
```

**Response:**
```json
{
  "answer": "Based on your emails, John mentioned the project deadline is next Friday...",
  "highlights": [
    {
      "id": "mem_123",
      "text": "The project deadline has been moved to next Friday",
      "source": "gmail",
      "metadata": {...}
    }
  ]
}
```

## Error Handling

Always handle potential errors:

```typescript
async function safeSearch(userId: string, query: string): Promise<string | null> {
  try {
    return await searchMemories(userId, query);
  } catch (error) {
    console.error('Memory search failed:', error);

    // Graceful degradation
    if (error instanceof Error && error.message.includes('unauthorized')) {
      return 'Unable to access memories. Please reconnect your accounts.';
    }

    return null; // Return null to indicate no results
  }
}
```

## Best Practices

1. **Cache Results**: For repeated queries, consider caching results
2. **Rate Limiting**: Be mindful of API rate limits in high-traffic applications
3. **Timeout Handling**: Set appropriate timeouts for API calls
4. **Fallback Behavior**: Have fallback behavior when memories aren't available
5. **User Feedback**: Let users know when memory search is being used

```typescript
// Example with caching
import { LRUCache } from 'lru-cache';

const cache = new LRUCache<string, string>({
  max: 100,
  ttl: 1000 * 60 * 5, // 5 minutes
});

async function searchWithCache(userId: string, query: string): Promise<string> {
  const cacheKey = `${userId}:${query}`;

  const cached = cache.get(cacheKey);
  if (cached) return cached;

  const result = await searchMemories(userId, query);
  cache.set(cacheKey, result);

  return result;
}
```
