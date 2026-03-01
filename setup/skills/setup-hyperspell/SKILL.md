---
name: Setting up Hyperspell
description: Guide the user through integrating Hyperspell into their project
allowed-tools: Bash, Read, Grep, Glob, Write, Edit, TodoWrite, AskUserQuestion
---

# Setup Hyperspell

## Instructions

Copy this checklist and track your progress:

```
Implementation Progress:
- [ ] Step 1: Install the Hyperspell SDK
- [ ] Step 2: Configure the API Key
- [ ] Step 3: Detect Project Type
- [ ] Step 4: Choose Memory Ingestion Method
- [ ] Step 5: Set Up Memory Ingestion
- [ ] Step 6: Set Up Memory Search
- [ ] Step 7: Wrapping Up
```

Run the following command and note the output as $START_TIME:

```sh
date -u +%Y-%m-%dT%H:%M:%SZ
```

### Step 1: Install the Hyperspell SDK

Determine whether the hyperspell SDK is already installed. If not, install it:
- For TypeScript/JavaScript projects: `npm i hyperspell` or `yarn add hyperspell`
- For Python projects: `uv add hyperspell` or `pip install hyperspell` (use whatever package manager the project uses)

### Step 2: Configure the API Key

When the user invoked this command, they may have passed an API key as an argument: '$1'

If that string is blank or not a valid API key, offer to create a new Hyperspell account automatically:

```
I can create a Hyperspell account for you automatically, or you can provide an existing API key.

Choose one:
1. Create new account (recommended - takes 30 seconds)
2. Use existing API key
```

#### Option 1: Create New Account (Bootstrap Flow)

If the user chooses to create a new account:

1. **Get their email address**: Ask "What's your email address?" (don't use multiple choice)

2. **Get the app name**: Ask "What should we call this Hyperspell app?" and suggest the current project name if detectable.

3. **Call the bootstrap API**: Make a POST request to `https://api.hyperspell.com/auth/bootstrap` with:
   ```json
   {
     "email": "user@example.com",
     "app_name": "Their App Name",
     "user_agent": "Claude Code 1.0"
   }
   ```

4. **Handle the response**: 
   - **Success**: Tell the user "Verification code sent to your email! Please check your inbox."
   - **Rate limited**: Show the error message and suggest manual signup at https://app.hyperspell.com/api-keys
   - **Other error**: Fall back to manual signup flow (Option 2)

5. **Get verification code**: Ask "Please enter the 6-digit code from the email:" (don't use multiple choice)

6. **Verify the code**: Make a POST request to `https://api.hyperspell.com/auth/bootstrap/verify` with:
   ```json
   {
     "verification_id": "from_step_3_response", 
     "code": "123456"
   }
   ```

7. **Handle verification response**:
   - **Success**: Use the returned `api_key` and continue to step 3
   - **Invalid code**: Let them retry (up to 3 times), then fall back to Option 2
   - **Other error**: Fall back to Option 2

#### Option 2: Use Existing API Key

If bootstrap fails or the user chooses to use an existing key:

```
Please paste your Hyperspell API key below.

If you don't have one yet, create one at https://app.hyperspell.com/api-keys
```

Do NOT use a multiple choice menu for this step - just ask directly and let the user paste their key.

#### Save the API Key

Once you have the API key (from either flow), put it in `.env.local` (or `.env` if `.env.local` doesn't exist) as `HYPERSPELL_API_KEY`.

If this project contains an `.env.example`, also put a dummy key in there (`HYPERSPELL_API_KEY=hs-0-xxxxxxxxxxxxxxxxxxxxxx`)

### Step 3: Detect Project Type

Follow the instructions in [detect_project.md](detection/detect_project.md) to analyze the codebase and determine:
- **Repo type**: Backend, Frontend-only, or Monorepo/Fullstack
- **Language**: TypeScript/JavaScript or Python
- **Framework**: Next.js (App/Pages Router), Express, FastAPI, SPA, etc.
- **Auth system**: Clerk, Auth0, NextAuth, custom, or none
- **AI SDK**: Vercel AI SDK, LangChain, or none

Store these detection results for use in subsequent steps.

### Step 4: Ask How User Wants to Add Memories

Display the following explanation to the user (replace `<YOUR PROJECT>` with the name of this project):

```
Hyperspell can create memories from many sources: email, Slack, documents, chat transcripts, or uploaded files.

Most projects let their users connect their accounts (Gmail, Slack, etc.) for automatic memory ingestion. Other projects add memories programmatically by uploading files or tracking conversations.

How do you want to create memories in <YOUR PROJECT>?
```

Ask the user with this multiple choice:
- **Let users connect their accounts** - Users authorize their Gmail, Slack, etc. and Hyperspell automatically ingests their data
- **Add memories directly** - You programmatically add memories via API (file uploads, conversation tracking, etc.)

### Step 5: Set Up Memory Ingestion

Based on the user's choice in Step 4 and the detection results from Step 3, set up the appropriate ingestion method.

---

**If user chose "Let users connect their accounts" (OAuth):**

The OAuth flow requires two pieces:
1. A **backend endpoint** that generates user tokens (using your API key)
2. A **frontend** that redirects users to `connect.hyperspell.com` with that token

Follow the instructions in [oauth.md](ingestion/oauth.md) and implement what applies to this project:

- **If the project has a backend:** Create the token endpoint using the framework examples in oauth.md.

- **If the project has NO backend (frontend-only):** Display this message:
  ```
  The OAuth connect flow requires a backend endpoint to securely generate user tokens. You'll need to create an endpoint on a separate backend that calls Hyperspell's /auth/user_token API, then have your frontend fetch from that endpoint.
  ```
  Then continue with the frontend setup, leaving a TODO placeholder for the token endpoint URL.

- **If the project has a frontend:** Create the connect button component using the React example in oauth.md.

- **If the project has NO frontend (backend-only):** Display this message:
  ```
  Since this is a backend-only project, you'll need to redirect users to the connect page from your external frontend. The token endpoint is ready - your frontend should fetch a token from it, then redirect to: https://connect.hyperspell.com?token={token}&redirect_uri={returnUrl}
  ```

---

**If user chose "Add memories directly" (Programmatic):**

- **If the project has a backend:** Follow the instructions in [index.md](ingestion/direct/index.md) to set up memory operations. Use the SDK with your API key and pass the user ID directly.

- **If the project has NO backend (frontend-only):** Display this message:
  ```
  Adding memories directly from a frontend requires a backend endpoint to securely make Hyperspell API calls. You'll need to either:
  1. Create a backend endpoint that proxies memory operations, OR
  2. Create a backend endpoint that generates user tokens (see oauth.md for examples)
  ```
  Then follow [index.md](ingestion/direct/index.md), noting that the user will need to set up the backend piece separately.

---

### Step 6: Set Up Memory Search (SDK Integration)

Display the following message:

```
Now that we've set up memory ingestion, let's integrate Hyperspell into your app so it can search and use those memories.
```

**Determine the integration pattern:**

First, analyze the codebase to see if the project has an existing AI/LLM integration with tools or function calling.

- **If the agent already uses tools** → Use **Pattern 1: Hyperspell as a Tool**. Do not ask, just proceed with this pattern.

- **If it's a simple conversational agent with no tools** → Ask the user which pattern they prefer (see below).

- **If there's no AI integration yet or it's unclear** → Ask the user which pattern they prefer (see below).

**When asking the user, present these options:**

```
How would you like to integrate Hyperspell's memory search?
```

- **As a tool in your AI calls** (Recommended) - Your AI decides when to search memories. Best for agents with multiple capabilities or when you want intelligent search decisions.

- **Direct search with AI answer** - Hyperspell answers questions directly from memories. Best for simple Q&A bots without other tools.

- **Direct search for context only** - Get relevant memory snippets to use however you want. Best for custom RAG pipelines or non-AI uses.

**Important:** Do not use "Direct search with AI answer" to replace an existing AI call if that call relies on other tools - those tools cannot be passed to the Hyperspell API.

**Based on the user's choice, follow the appropriate section in [index.md](search/index.md):**
- "As a tool" → Create the search helper, then follow the tool integration for their SDK (Vercel AI, OpenAI, Anthropic, etc.)
- "Direct search with AI answer" → Create the search helper only, call it with `answer: true` from existing code. Do NOT create a tool wrapper.
- "Direct search for context only" → Create the search helper only, call it with `answer: false` from existing code. Do NOT create a tool wrapper.

### Step 7: Wrapping Up

Run the following command:

```sh
date -u +%Y-%m-%dT%H:%M:%SZ
```

Compare the output with $START_TIME and calculate how many minutes have passed as DURATION.

Display the following message (replace <DURATION> with the actual duration):

```
Congratulations! You've successfully integrated Hyperspell in just <DURATION> minutes!

This is just the beginning of your journey with Hyperspell. As your project grows, Hyperspell grows with you. If you ever need help, you can use the /hyperspell:help command to get a direct line to the founders, right here from Claude Code.
```
