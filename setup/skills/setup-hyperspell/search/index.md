# Search Integration

This guide covers how to integrate Hyperspell's memory search into your application.

**Important:** Create a **search helper function**, then integrate it into your existing AI calls. Do NOT create new API endpoints.

## Step 1: Create the Search Helper

First, add the search helper to your Hyperspell utilities file (e.g., `lib/hyperspell.ts`):

```typescript
// lib/hyperspell.ts
import Hyperspell from 'hyperspell';

export async function searchMemories(
  userId: string,
  query: string,
  options: { answer?: boolean } = {}
) {
  const { answer = true } = options;

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await hyperspell.memories.search({
    query,
    answer,
  });

  return response;
}
```

```python
# lib/hyperspell.py
import os
from hyperspell import Hyperspell

def search_memories(user_id: str, query: str, answer: bool = True):
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    response = client.memories.search(
        query=query,
        answer=answer
    )

    return response
```

## Step 2: Integrate with Your AI

Now integrate the search helper based on how your app uses AI:

---

### If using Vercel AI SDK

See [vercel_ai.md](vercel_ai.md) for detailed instructions on creating a tool wrapper and adding it to `streamText`/`generateText`.

---

### If using OpenAI client directly

Add Hyperspell as a function/tool in your OpenAI calls:

```typescript
import OpenAI from 'openai';
import { searchMemories } from '@/lib/hyperspell';

const openai = new OpenAI();

const tools = [{
  type: 'function' as const,
  function: {
    name: 'search_memories',
    description: 'Search user memories including emails, documents, and messages. Use before answering questions about personal or work data.',
    parameters: {
      type: 'object',
      properties: {
        query: { type: 'string', description: 'The search query' },
      },
      required: ['query'],
    },
  },
}];

// In your chat handler
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages,
  tools,
});

// Handle tool calls
if (response.choices[0].message.tool_calls) {
  for (const toolCall of response.choices[0].message.tool_calls) {
    if (toolCall.function.name === 'search_memories') {
      const { query } = JSON.parse(toolCall.function.arguments);
      const result = await searchMemories(userId, query);
      // Add tool result to messages and continue conversation
    }
  }
}
```

---

### If using Anthropic client directly

Add Hyperspell as a tool in your Anthropic calls:

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { searchMemories } from '@/lib/hyperspell';

const anthropic = new Anthropic();

const tools = [{
  name: 'search_memories',
  description: 'Search user memories including emails, documents, and messages. Use before answering questions about personal or work data.',
  input_schema: {
    type: 'object',
    properties: {
      query: { type: 'string', description: 'The search query' },
    },
    required: ['query'],
  },
}];

// In your chat handler
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-20250514',
  messages,
  tools,
});

// Handle tool use
for (const block of response.content) {
  if (block.type === 'tool_use' && block.name === 'search_memories') {
    const result = await searchMemories(userId, block.input.query);
    // Add tool result to messages and continue conversation
  }
}
```

---

### If using LangChain

```typescript
import { DynamicTool } from 'langchain/tools';
import { searchMemories } from '@/lib/hyperspell';

function createMemoriesTool(userId: string) {
  return new DynamicTool({
    name: 'search_memories',
    description: 'Search user memories including emails, documents, and messages.',
    func: async (query: string) => {
      const response = await searchMemories(userId, query);
      return response.answer ?? 'No relevant information found.';
    },
  });
}
```

---

### If there are no existing AI calls

If your app doesn't have any AI/LLM integration yet, just create the search helper for now.

Display this message to the user:

```
I've created the searchMemories helper in lib/hyperspell.ts. You can use it to search through connected memories:

  const result = await searchMemories(userId, "What did John say about the deadline?");
  // result.answer - AI-generated answer from memories (when answer: true)
  // result.documents - source documents that matched the query

When you add AI/chat functionality later, you can integrate this as a tool so your AI can automatically search memories when relevant.
```

## Tool Description Best Practices

Good descriptions help the AI use the tool effectively:

```
"Search through the user's connected memories including emails, Slack messages,
and documents. Use this tool BEFORE answering questions that might require
information from the user's personal or work data. Phrase your query as a
question for best results."
```
