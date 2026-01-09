# Project Detection

Analyze the codebase to determine its architecture and technology stack. This information is critical for selecting the correct auth flow and integration patterns.

## Detection Process

Examine the project structure to determine:

### 1. Repo Type

Determine if this is a **Backend**, **Frontend-only**, or **Monorepo/Fullstack** project:

**Backend indicators:**
- Has server-side entry points (e.g., `server.js`, `app.py`, `main.py`)
- Uses backend frameworks: Express, Fastify, Koa, Flask, FastAPI, Django, NestJS
- Has API routes without a frontend build system
- No `index.html` or frontend framework in root

**Frontend-only indicators:**
- Pure SPA with no server code (React, Vue, Svelte, Angular)
- Uses Vite, Create React App, or similar frontend-only tooling
- Has `index.html` as entry point
- No API routes or server-side code
- May use services like Vercel/Netlify for hosting

**Monorepo/Fullstack indicators:**
- Next.js project (has both frontend and API routes)
- Has separate `packages/`, `apps/` directories for frontend and backend
- Remix, Nuxt, SvelteKit projects
- Has both client and server code in same repo

### 2. Language

Determine primary language:
- **TypeScript/JavaScript**: Check for `package.json`, `.ts`/`.js` files, `tsconfig.json`
- **Python**: Check for `requirements.txt`, `pyproject.toml`, `setup.py`, `.py` files

### 3. Framework

Identify the specific framework by checking:

**JavaScript/TypeScript:**
- `next` in package.json → **Next.js** (then check for `app/` vs `pages/` directory)
- `express` in package.json → **Express**
- `fastify` in package.json → **Fastify**
- `@nestjs/core` in package.json → **NestJS**
- `react` without Next.js → **React SPA**
- `vue` → **Vue SPA**
- `svelte` or `@sveltejs/kit` → **Svelte/SvelteKit**

**Python:**
- `fastapi` in dependencies → **FastAPI**
- `flask` in dependencies → **Flask**
- `django` in dependencies → **Django**

### 4. Auth System

Check for existing authentication:
- `@clerk/` packages → **Clerk**
- `@auth0/` packages → **Auth0**
- `next-auth` → **NextAuth**
- `firebase` with auth imports → **Firebase Auth**
- Custom JWT/session handling → **Custom**
- No auth detected → **None**

### 5. AI SDK

Check for AI/agent SDKs:
- `ai` package (from Vercel) → **Vercel AI SDK**
- `langchain` → **LangChain**
- `llamaindex` → **LlamaIndex**
- `@anthropic-ai/sdk` → **Anthropic SDK**
- `openai` → **OpenAI SDK**
- None detected → **None/Custom**

## Output Format

After detection, summarize findings:

```
Project Detection Results:
- Repo Type: [Backend | Frontend-only | Monorepo/Fullstack]
- Language: [TypeScript/JavaScript | Python]
- Framework: [Next.js App Router | Next.js Pages Router | Express | FastAPI | React SPA | etc.]
- Auth System: [Clerk | Auth0 | NextAuth | Firebase | Custom | None]
- AI SDK: [Vercel AI SDK | LangChain | None/Custom]
```

## Detection Commands

Use these commands to gather information:

```sh
# Check for package.json (Node.js projects)
cat package.json 2>/dev/null | head -100

# Check for Python dependencies
cat requirements.txt 2>/dev/null || cat pyproject.toml 2>/dev/null | head -50

# Check directory structure
ls -la
ls -la src/ 2>/dev/null
ls -la app/ 2>/dev/null
ls -la pages/ 2>/dev/null
ls -la packages/ 2>/dev/null
```

## Important Notes

- For **Next.js**, distinguish between App Router (`app/` directory) and Pages Router (`pages/` directory)
- For **Monorepos**, identify which package/app is the primary target for Hyperspell integration
- If the project type is ambiguous, ask the user for clarification
- Store detection results to pass to subsequent skill files
