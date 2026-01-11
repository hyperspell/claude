# Search Integration

This guide covers how to query memories from Hyperspell in your application.

## Two Integration Patterns

### Pattern 1: Hyperspell as a Tool (Most Common)

Wrap the memory search in a tool that you pass to your AI/LLM. The AI decides when to search memories.

```
User Question → Your AI (with Hyperspell tool) → AI calls tool when needed → Response
```

**When to use:**
- Your agent already uses tools/function calling
- You want the AI to decide when memory context is relevant
- You have other tools alongside memories

**Implementation:** See [vercel_ai.md](vercel_ai.md) or the adaptation guidance below.

### Pattern 2: Direct Memory Search

Call the memory search API directly and use the results in your application.

**With `answer: true`:**
- Hyperspell's AI generates an answer based on the user's memories
- Returns both the answer AND source documents
- Good for: Simple Q&A bots without other tools

**With `answer: false`:**
- Returns only the relevant source documents/highlights
- You inject this context into your own AI call
- Good for: Custom RAG pipelines, non-AI uses

```
User Question → Hyperspell Search → Context/Answer → Your app
```

**Implementation:** See [raw_api.md](raw_api.md).

## Choosing the Right Pattern

**Use Pattern 1 (Tool) if:**
- Your agent already has tools → **Definitely use Pattern 1**
- You need the AI to intelligently decide when to search
- You have multiple tools the AI should coordinate

**Use Pattern 2 (Direct) if:**
- Simple Q&A bot with no other tools
- You want Hyperspell to fully handle the AI response (`answer: true`)
- You need raw context for custom processing (`answer: false`)

## Basic Search Code

This is the core search call - use it wherever your backend logic lives:

```typescript
import Hyperspell from 'hyperspell';

async function searchMemories(userId: string, query: string) {
  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await hyperspell.memories.search({
    query,
    answer: true,  // or false for raw context
  });

  return response;
}
```

```python
from hyperspell import Hyperspell
import os

def search_memories(user_id: str, query: str):
    hyperspell = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    response = hyperspell.memories.search(
        query=query,
        answer=True  # or False for raw context
    )

    return response
```

## SDK Routing

Based on your AI SDK:

- **Vercel AI SDK** (`ai` package) → [vercel_ai.md](vercel_ai.md)
- **No SDK / Direct API** → [raw_api.md](raw_api.md)
- **Other SDKs** (LangChain, LlamaIndex, OpenAI, Anthropic) → See below

## Adaptation Guidance for Other SDKs

The pattern is the same for any SDK: create a tool that calls `hyperspell.memories.search()`.

### LangChain

```typescript
import { DynamicTool } from 'langchain/tools';
import Hyperspell from 'hyperspell';

function createMemoriesTool(userId: string) {
  return new DynamicTool({
    name: 'search_memories',
    description: 'Search user memories including emails, documents, and messages.',
    func: async (query: string) => {
      const hyperspell = new Hyperspell({
        apiKey: process.env.HYPERSPELL_API_KEY!,
        userID: userId,
      });
      const response = await hyperspell.memories.search({ query, answer: true });
      return response.answer ?? 'No relevant information found.';
    },
  });
}
```

### OpenAI Function Calling

```typescript
const tools = [{
  type: 'function',
  function: {
    name: 'search_memories',
    description: 'Search user memories including emails, documents, and messages',
    parameters: {
      type: 'object',
      properties: {
        query: { type: 'string', description: 'The search query' },
      },
      required: ['query'],
    },
  },
}];

// In your tool handler:
if (toolCall.function.name === 'search_memories') {
  const { query } = JSON.parse(toolCall.function.arguments);
  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });
  const response = await hyperspell.memories.search({ query, answer: true });
  return response.answer;
}
```

### Anthropic Claude

```typescript
const tools = [{
  name: 'search_memories',
  description: 'Search user memories including emails, documents, and messages',
  input_schema: {
    type: 'object',
    properties: {
      query: { type: 'string', description: 'The search query' },
    },
    required: ['query'],
  },
}];

// In your tool handler:
if (toolUse.name === 'search_memories') {
  const { query } = toolUse.input;
  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });
  const response = await hyperspell.memories.search({ query, answer: true });
  return response.answer;
}
```

## Tool Description Best Practices

Good descriptions help the AI use the tool effectively:

```
"Search through the user's connected memories including emails, Slack messages,
and documents. Use this tool BEFORE answering questions that might require
information from the user's personal or work data. Phrase your query as a
question for best results."
```
