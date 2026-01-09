# SDK Integration

This guide helps you integrate Hyperspell memory search into your AI agent or application.

## Two Integration Patterns

There are two core ways to use Hyperspell's memory search:

### Pattern 1: Hyperspell as a Tool (Most Common)

Wrap the memory search API in a tool that you pass to your existing AI/LLM call. The AI decides when to search memories.

```
User Question → Your AI (with Hyperspell tool) → AI calls tool when needed → Response
```

**When to use:**
- Your agent already uses tools/function calling
- You're using LLMs for intelligent function routing
- You want the AI to decide when memory context is relevant
- You have other tools the AI needs access to alongside memories

### Pattern 2: Direct Memory Search

Call the memory search API directly and use the results in your application.

**With `answer: true`:**
- Hyperspell's AI generates an answer based on the user's memories
- Returns both the answer AND source documents
- Good for: Conversational agents that don't use other tools

**With `answer: false`:**
- Returns only the relevant source documents/highlights
- You inject this context into your own AI call or use it directly
- Good for: Custom context injection, RAG pipelines, non-AI uses

```
User Question → Hyperspell Search → Context/Answer → Your app (or your own AI call)
```

**When to use:**
- Simple conversational agent with no other tools
- You want to control exactly how context is used
- Building a RAG pipeline with custom prompting
- Non-AI use cases (search UI, context display)

## Choosing the Right Pattern

**Use Pattern 1 (Tool) if:**
- Your agent already has tools → **Definitely use Pattern 1**
- You need the AI to intelligently decide when to search
- You have multiple tools the AI should coordinate

**Use Pattern 2 (Direct) if:**
- Simple Q&A bot with no other tools
- You want Hyperspell to fully handle the AI response (`answer: true`)
- You need raw context for custom processing (`answer: false`)

**If unclear, ask the user:**
```
How would you like to integrate Hyperspell's memory search?

1. **As a tool in your AI calls** (Recommended) - Your AI decides when to search memories. Best for agents with multiple capabilities.

2. **Direct search with AI answer** - Hyperspell answers questions directly from memories. Best for simple Q&A without other tools.

3. **Direct search for context only** - Get relevant memory snippets to use however you want. Best for custom RAG or non-AI uses.
```

## SDK Routing

Based on the detected AI SDK and chosen pattern:

- **Vercel AI SDK** (`ai` package) → `./vercel_ai.md`
- **No SDK / Direct API** → `./raw_api.md`
- **Other SDKs** (LangChain, LlamaIndex, etc.) → See Adaptation Guidance below

## Search Function Template

The core search function (adapt to your framework):

```typescript
async function searchMemories(userId: string, query: string): Promise<string> {
  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await hyperspell.memories.search({
    query,
    answer: true,
    sources: ['gmail', 'slack'], // Your configured providers
  });

  return response.answer;
}
```

## Adaptation Guidance

### LangChain

Create a custom tool for LangChain:

```typescript
import { DynamicTool } from 'langchain/tools';
import Hyperspell from 'hyperspell';

function createMemoriesTool(userId: string) {
  return new DynamicTool({
    name: 'search_memories',
    description: 'Search through user memories including emails, documents, and messages. Use this to find information the user has stored.',
    func: async (query: string) => {
      const hyperspell = new Hyperspell({
        apiKey: process.env.HYPERSPELL_API_KEY!,
        userID: userId,
      });

      const response = await hyperspell.memories.search({
        query,
        answer: true,
      });

      return response.answer;
    },
  });
}

// Usage with an agent
const tools = [createMemoriesTool(userId)];
const agent = createReactAgent({ llm, tools, prompt });
```

### LlamaIndex

Create a query tool for LlamaIndex:

```typescript
import { FunctionTool } from 'llamaindex';
import Hyperspell from 'hyperspell';

function createMemoriesTool(userId: string) {
  return FunctionTool.from(
    async ({ query }: { query: string }) => {
      const hyperspell = new Hyperspell({
        apiKey: process.env.HYPERSPELL_API_KEY!,
        userID: userId,
      });

      const response = await hyperspell.memories.search({
        query,
        answer: true,
      });

      return response.answer;
    },
    {
      name: 'search_memories',
      description: 'Search through user memories to find relevant information',
      parameters: {
        type: 'object',
        properties: {
          query: {
            type: 'string',
            description: 'The search query - phrase it as a question',
          },
        },
        required: ['query'],
      },
    }
  );
}
```

### OpenAI Function Calling

Define as an OpenAI function:

```typescript
import OpenAI from 'openai';
import Hyperspell from 'hyperspell';

const openai = new OpenAI();

const tools: OpenAI.ChatCompletionTool[] = [
  {
    type: 'function',
    function: {
      name: 'search_memories',
      description: 'Search through user memories including emails, documents, and messages',
      parameters: {
        type: 'object',
        properties: {
          query: {
            type: 'string',
            description: 'The search query',
          },
        },
        required: ['query'],
      },
    },
  },
];

async function handleToolCall(userId: string, toolCall: OpenAI.ChatCompletionMessageToolCall) {
  if (toolCall.function.name === 'search_memories') {
    const args = JSON.parse(toolCall.function.arguments);
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
      userID: userId,
    });
    const response = await hyperspell.memories.search({
      query: args.query,
      answer: true,
    });
    return response.answer;
  }
  throw new Error(`Unknown tool: ${toolCall.function.name}`);
}

// In your chat completion loop:
const response = await openai.chat.completions.create({
  model: 'gpt-4',
  messages,
  tools,
});

if (response.choices[0].message.tool_calls) {
  for (const toolCall of response.choices[0].message.tool_calls) {
    const result = await handleToolCall(userId, toolCall);
    // Add result to messages and continue
  }
}
```

### Anthropic Claude

For direct Claude API usage:

```typescript
import Anthropic from '@anthropic-ai/sdk';
import Hyperspell from 'hyperspell';

const anthropic = new Anthropic();

const tools: Anthropic.Tool[] = [
  {
    name: 'search_memories',
    description: 'Search through user memories including emails, documents, and messages',
    input_schema: {
      type: 'object',
      properties: {
        query: {
          type: 'string',
          description: 'The search query',
        },
      },
      required: ['query'],
    },
  },
];

async function handleToolUse(userId: string, toolUse: Anthropic.ToolUseBlock) {
  if (toolUse.name === 'search_memories') {
    const input = toolUse.input as { query: string };
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
      userID: userId,
    });
    const response = await hyperspell.memories.search({
      query: input.query,
      answer: true,
    });
    return response.answer;
  }
  throw new Error(`Unknown tool: ${toolUse.name}`);
}
```

## Best Practices

1. **Tool Description**: Write clear descriptions that help the AI know when to use the memory search
2. **Query Formatting**: Instruct the AI to phrase queries as questions for better results
3. **Error Handling**: Always handle potential API errors gracefully
4. **User Context**: Always pass the correct user ID to scope searches appropriately
5. **Caching**: Consider caching frequent queries to reduce API calls

## Tool Description Examples

Good descriptions help the AI use the tool effectively:

```
"Search through the user's connected memories including emails, Slack messages,
and documents. Use this tool BEFORE answering questions that might require
information from the user's personal or work data. Phrase your query as a
question for best results."
```

```
"Access the user's knowledge base. Use this to find information from their
emails, documents, chat history, and other connected sources. Always check
memories when the user asks about specific people, projects, or events."
```
