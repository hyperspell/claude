# Search Integration

This guide covers how to integrate Hyperspell's memory search into your application.

**Important:** Create a **search helper function**, then integrate it into your existing code. Do NOT create new API endpoints.

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

## Step 2: Choose Your Integration Pattern

Based on the user's choice from the SKILL.md options, follow the appropriate pattern below:

---

## Pattern: As a Tool (for AI with tools/function calling)

If the user chose "As a tool in your AI calls", wrap the search helper in a tool for their AI SDK.

### Vercel AI SDK

See [vercel_ai.md](vercel_ai.md) for detailed instructions.

### OpenAI client

```typescript
import OpenAI from 'openai';
import { searchMemories } from '@/lib/hyperspell';

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

// Add tools to your existing chat call
const response = await openai.chat.completions.create({
  model: 'gpt-4o',
  messages,
  tools,
});

// Handle tool calls in your existing handler
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

### Anthropic client

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { searchMemories } from '@/lib/hyperspell';

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

// Add tools to your existing chat call
const response = await anthropic.messages.create({
  model: 'claude-sonnet-4-20250514',
  messages,
  tools,
});

// Handle tool use in your existing handler
for (const block of response.content) {
  if (block.type === 'tool_use' && block.name === 'search_memories') {
    const result = await searchMemories(userId, block.input.query);
    // Add tool result to messages and continue conversation
  }
}
```

---

## Pattern: Direct Search with AI Answer

If the user chose "Direct search with AI answer", do NOT create a tool wrapper. Instead, call the search helper directly with `answer: true` and inject the answer as context into the existing AI call.

This pattern lets Hyperspell's AI answer questions from memories, then passes that answer as context to your existing AI:

```typescript
// In your existing AI/chat code
import { searchMemories } from '@/lib/hyperspell';

async function handleWithMemoryContext(userId: string, userMessage: string) {
  // Get Hyperspell's AI answer from memories
  const result = await searchMemories(userId, userMessage, { answer: true });

  // Inject as context into your existing AI call
  const memoryContext = result.answer
    ? `Information from user's memories:\n${result.answer}`
    : '';

  // Continue with your existing AI call, adding the context...
  // e.g., prepend to system prompt or add as a message
}
```

Look for existing places in the codebase where AI calls are made (chat handlers, completion endpoints, etc.) and inject the memory context there.

**If there's no existing AI call to integrate with:**

Just create the search helper and display this message:

```
I've created the searchMemories helper in lib/hyperspell.ts. You can use it to search through connected memories:

  const result = await searchMemories(userId, "What did John say about the deadline?");
  // result.answer - AI-generated answer from memories (when answer: true)
  // result.documents - source documents that matched the query

When you add AI/chat functionality later, you can inject result.answer as context into your AI calls.
```

---

## Pattern: Direct Search for Context Only

If the user chose "Direct search for context only", do NOT create a tool wrapper. Instead, call the search helper with `answer: false` to get raw context.

This pattern returns source documents without an AI answer. **Only integrate this into existing code if there's an obvious place** (e.g., a clear RAG pipeline or context-injection point). In most cases, there won't be an obvious place.

**Otherwise (most common):**

Just create the search helper and display a message like this, updated based on context (eg. if the project is in Python, include a Python example instead of javascript):

```
I've created the searchMemories helper in lib/hyperspell.ts. You can use it to search through connected memories:

  const result = await searchMemories(userId, "What did John say about the deadline?", { answer: false });
  // result.documents - source documents that matched the query

To use this in your AI calls, build context from the documents and inject it into your prompt:

  const context = result.documents
    .map(doc => doc.title ? `${doc.title}: ${doc.text}` : doc.text)
    .join('\\n\\n');
```

---

## Tool Description Best Practices

When using the tool pattern, good descriptions help the AI use the tool effectively:

```
"Search through the user's connected memories including emails, Slack messages,
and documents. Use this tool BEFORE answering questions that might require
information from the user's personal or work data. Phrase your query as a
question for best results."
```
