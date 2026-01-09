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
- [ ] Step 5: Determine Auth Flow
- [ ] Step 6: Set Up Memory Ingestion
- [ ] Step 7: Set Up Memory Search
- [ ] Step 8: Wrapping Up
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

If that string is blank or not a valid API key, display this message and wait for the user to paste their key:

```
Please paste your Hyperspell API key below.

If you don't have one yet, create one at https://app.hyperspell.com/api-keys
```

Do NOT use a multiple choice menu for this step - just ask directly and let the user paste their key.

Once you have the API key, put it in `.env.local` (or `.env` if `.env.local` doesn't exist) as `HYPERSPELL_API_KEY`.

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

### Step 5: Determine Auth Flow

Based on the user's choice in Step 4 and the detection results from Step 3, determine which auth flow to use:

**If user chose "Let users connect their accounts" (OAuth):**
- If repo has a frontend (Monorepo/Fullstack or Frontend-only with backend endpoint): **FLOW A**
  - Backend creates endpoint for user tokens
  - Frontend calls backend, then redirects to connect.hyperspell.com
- If repo is backend-only: **FLOW B**
  - Backend generates tokens and serves/redirects to connect page

**If user chose "Add memories directly" (Programmatic):**
- If repo has a backend: **FLOW C**
  - Use API key with `X-As-User` header (simplest)
- If repo is frontend-only: **FLOW D**
  - Need external token endpoint (Clerk/Auth0 or separate backend)

### Step 6: Set Up Memory Ingestion

**For OAuth flows (FLOW A or B):**
1. Follow instructions in [connect_oauth.md](memories/connect_oauth.md) for SDK initialization
2. Then follow [index.md](frameworks/index.md) to set up the framework-specific token endpoint and connect button

**For Direct memory flows (FLOW C or D):**
1. Follow instructions in [index.md](memories/direct/index.md) to set up memory operations
2. If FLOW D (frontend-only), also follow [frontend_only.md](frameworks/frontend_only.md) for token setup

### Step 7: Set Up Memory Search (SDK Integration)

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

Follow instructions in [index.md](sdks/index.md) to implement the chosen pattern.

### Step 8: Wrapping Up

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

## Auth Flow Reference

| Flow | Scenario | Auth Method |
|------|----------|-------------|
| A | Monorepo/Fullstack + OAuth | Backend token endpoint → Frontend connect button |
| B | Backend-only + OAuth | Backend token endpoint → Serve connect page |
| C | Backend + Direct | API key + X-As-User header |
| D | Frontend-only | External token endpoint required |
