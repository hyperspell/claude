# Vercel AI SDK Integration

This guide covers integrating Hyperspell with the Vercel AI SDK (`ai` package) using **Pattern 1: Hyperspell as a Tool**.

## When to Use This Pattern

Use this tool-based approach when:
- Your agent already uses tools/function calling
- You want the AI to decide when to search memories
- You have other tools alongside Hyperspell

If you want Hyperspell to directly answer questions (without going through your AI), see [raw_api.md](raw_api.md) for the direct search patterns.

## Prerequisites

Make sure you have:
1. The Vercel AI SDK installed (`npm i ai`)
2. Hyperspell server actions or API routes set up (see [frameworks](../frameworks/index.md) guides)
3. Zod installed (`npm i zod`)

## Create the Memories Tool

Find the appropriate place to add tools in your project. This could be:
- A dedicated tools file (e.g., `lib/tools.ts` or `tools/index.ts`)
- The same file that makes the `streamText`/`generateText` call
- A tools directory if you have multiple tools

Add the following code:

```typescript
// lib/tools/memories.ts (or wherever you keep tools)
import { tool } from 'ai';
import { z } from 'zod';

// Option 1: If using server actions (Next.js App Router)
import { searchMemories } from '@/app/actions/hyperspell';

export const memories = tool({
  description: 'Search connected memories for information including emails, documents, and messages. ALWAYS use this tool before answering questions that might require information from the user\'s personal or work data.',
  inputSchema: z.object({
    query: z.string().describe('The search query. Formulate it as a question for best results.'),
  }),
  execute: async ({ query }) => {
    const result = await searchMemories(query);
    return result;
  },
});
```

Or if using API routes:

```typescript
// lib/tools/memories.ts
import { tool } from 'ai';
import { z } from 'zod';

export const memories = tool({
  description: 'Search connected memories for information including emails, documents, and messages. ALWAYS use this tool before answering questions that might require information from the user\'s personal or work data.',
  inputSchema: z.object({
    query: z.string().describe('The search query. Formulate it as a question for best results.'),
  }),
  execute: async ({ query }) => {
    // Call your API route
    const response = await fetch('/api/hyperspell/search', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query }),
    });
    const data = await response.json();
    return data.answer ?? 'No relevant information found in memories.';
  },
});
```

**IMPORTANT**: Follow the exact implementation above, do not use different arguments. Define tools exactly as show above.

## Add the Tool to Your Agent

Find where you call `streamText` or `generateText` and add the memories tool:

### With streamText (Streaming)

```typescript
import { streamText, stepCountIs } from 'ai';
import { openai } from '@ai-sdk/openai';
import { memories } from '@/lib/tools/memories';

const result = await streamText({
  model: openai('gpt-4o'),
  messages,
  tools: {
    memories,
    // ... other tools you may have
  },
  stopWhen: stepCountIs(5), // Allow enough steps for tool use
});
```

### With generateText (Non-streaming)

```typescript
import { generateText, stepCountIs } from 'ai';
import { openai } from '@ai-sdk/openai';
import { memories } from '@/lib/tools/memories';

const result = await generateText({
  model: openai('gpt-4o'),
  messages,
  tools: {
    memories,
  },
  stopWhen: stepCountIs(5),
});
```

## Full Example: Chat API Route

Here's a complete example for a Next.js App Router API route:

```typescript
// app/api/chat/route.ts
import { streamText, stepCountIs, tool } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';
import Hyperspell from 'hyperspell';
import { auth } from '@/lib/auth';

export async function POST(req: Request) {
  const session = await auth();
  const userId = session?.user?.id || 'anonymous';

  const { messages } = await req.json();

  // Create the memories tool with user context
  const memories = tool({
    description: 'Search connected memories for information. Use this before answering questions that might require personal or work data.',
    inputSchema: z.object({
      query: z.string().describe('The search query as a question'),
    }),
    execute: async ({ query }) => {
      const hyperspell = new Hyperspell({
        apiKey: process.env.HYPERSPELL_API_KEY!,
        userID: userId,
      });

      const response = await hyperspell.memories.search({
        query,
        answer: true,
      });

      return response.answer ?? 'No relevant information found in memories.';
    },
  });

  const result = streamText({
    model: openai('gpt-4o'),
    system: `You are a helpful assistant with access to the user's memories.
When the user asks questions that might be answered by their emails,
documents, or messages, use the memories tool to search for relevant information.`,
    messages,
    tools: { memories },
    stopWhen: stepCountIs(5),
  });

  return result.toDataStreamResponse();
}
```

## Important Configuration

### stopWhen

Always set `stopWhen` to allow enough steps for tool use (at least 5, or higher for complex tool chains):

```typescript
import { stepCountIs } from 'ai';

streamText({
  // ...
  stopWhen: stepCountIs(5),  // Required for tool use
});
```

If `stopWhen` is already set to a higher step count, don't reduce it.

### Tool Choice

You can control when tools are used:

```typescript
streamText({
  // ...
  tools: { memories },
  toolChoice: 'auto',  // Let the model decide (default)
  // OR
  toolChoice: 'required',  // Force tool use
  // OR
  toolChoice: { type: 'tool', toolName: 'memories' },  // Use specific tool
});
```

## With Multiple Tools

If you already have tools, add memories alongside them:

```typescript
import { streamText, stepCountIs } from 'ai';
import { memories } from '@/lib/tools/memories';
import { calculator } from '@/lib/tools/calculator';
import { webSearch } from '@/lib/tools/webSearch';

streamText({
  model: openai('gpt-4o'),
  messages,
  tools: {
    memories,
    calculator,
    webSearch,
  },
  stopWhen: stepCountIs(10),  // Increase for more complex interactions
});
```

## Troubleshooting

### Tool not being called
- Check that the tool description clearly explains when to use it
- Verify `stopWhen` step count is high enough
- Add explicit instructions in your system prompt

### Search returning empty
- Verify the user has connected accounts via Hyperspell Connect
- Check that the user ID is being passed correctly
- Ensure the API key has the correct permissions

### Type errors
- Make sure `zod` is installed
- Verify import paths are correct for your project structure
