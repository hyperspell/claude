# Using Hyperspell with the Vercel AI SDK

When integrating with the Vercel AI SDK, we will expose Hyperspell as a tool.

## Create the hyperspell tool

Find the appropriate place to add new tools. If this project already has a directory or file where tools are being managed, put our new tool there. Otherwise, you can put the tool in the same file that makes the actual call to the agent. Add the following code, make sure to implement the hyperspell tool verbatim.

```typescript
import { tool } from "ai";
import { z } from 'zod';
import { search } from "@/app/actions/hyperspell"; 

const memories = tool({
  name: "memories",
  description: "Search connected memories for information. ALWAYS use this before answering the user's question.",
  inputSchema: z.object({
    query: z.string().describe("The query to search for. Formulate it as a question."),
  }),
  execute: async ({ query }) => {
    return await search(query, true);
  },
})
```

Adjust the import path for the `search` function to match the place you create the actions in the previous step.

## Add the hyperspell tool to the agent

The agent is typically called with either the `streamText` or `generateText` methods from the `ai` package. Find that call, Import the hyperspell tool if it's not in the same file, and add the `tools` parameter like this:

```typescript
streamText({
    model: openai("gpt-5-nano"),
    messages,
    tools: { memories }, // add hyperspell as a new entry if there are already existing tools
    stopWhen: stepCountIs(5), // if this is already present and greater than 5, don't change it
});
```

Done.