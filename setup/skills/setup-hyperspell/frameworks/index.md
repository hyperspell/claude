# Framework Integration

This guide helps you set up Hyperspell integration for your specific framework. The implementation depends on the auth flow determined in the main orchestrator.

## Auth Flow Reference

| Flow | Scenario | What to Implement |
|------|----------|-------------------|
| A | Monorepo/Fullstack + OAuth | Backend token endpoint + Frontend connect button |
| B | Backend-only + OAuth | Backend token endpoint + Serve connect page |
| C | Backend + Direct | X-As-User header pattern (no endpoints needed) |
| D | Frontend-only | External token endpoint (see [frontend_only.md](frontend_only.md)) |

## Framework Routing

Based on the detected framework, follow the appropriate guide:

### JavaScript/TypeScript Frameworks

- **Next.js App Router** → [nextjs_app_router.md](nextjs_app_router.md)
- **Next.js Pages Router** → [nextjs_pages_router.md](nextjs_pages_router.md)
- **Express.js** → [express.md](express.md)
- **Fastify** → Adapt from Express pattern (similar middleware approach)
- **NestJS** → Adapt from Express pattern (use decorators and services)

### Python Frameworks

- **FastAPI** → [fastapi.md](fastapi.md)
- **Flask** → Adapt from FastAPI pattern (use blueprints instead of routers)
- **Django** → Adapt from FastAPI pattern (use views and URL patterns)

### Frontend-only (SPA)

- **React/Vue/Svelte SPA** → [frontend_only.md](frontend_only.md)

## Adaptation Guidance

If your framework isn't listed, follow these patterns:

### For OAuth Flows (A or B) - What You Need to Implement

1. **Token Endpoint**: Create an API endpoint that:
   - Authenticates the current user (using your auth system)
   - Calls Hyperspell's `/auth/user_token` with your API key and the user's ID
   - Returns the token to the client

   ```
   POST /api/hyperspell/token
   → Returns: { token: "hs-user-..." }
   ```

2. **Connect Button** (Flow A only): Add a UI element that:
   - Fetches a token from your token endpoint
   - Redirects to `https://connect.hyperspell.com?token={token}&redirect_uri={currentUrl}`

3. **Search Endpoint** (optional but recommended): Create an endpoint that:
   - Accepts a search query
   - Calls Hyperspell's search API
   - Returns results to the client

### For Direct Flows (C) - What You Need

Just use the X-As-User header pattern in your existing code:

```typescript
const client = new Hyperspell({
  apiKey: process.env.HYPERSPELL_API_KEY,
  userID: currentUserId,
});
```

No new endpoints needed - just call Hyperspell directly from your backend code.

### For Frontend-only (D)

See [frontend_only.md](frontend_only.md) for handling the special case where you have no backend.

## Common Patterns Across Frameworks

### Getting the Current User ID

Every framework has its own way to get the authenticated user. Examples:

```typescript
// Next.js with NextAuth
import { getServerSession } from 'next-auth';
const session = await getServerSession(authOptions);
const userId = session?.user?.id;

// Express with Passport
const userId = req.user?.id;

// FastAPI with dependency injection
async def get_user_id(token: str = Depends(oauth2_scheme)):
    return decode_token(token).user_id
```

Adapt to your auth system - the key is getting a stable user identifier.

### Environment Variables

All frameworks should read the API key from environment:

```
HYPERSPELL_API_KEY=hs-0-your-api-key
```

Never expose this key to the frontend - it should only be used server-side.

### Error Handling

Always handle Hyperspell API errors gracefully:

```typescript
try {
  const result = await hyperspell.memories.search({ query });
  return result;
} catch (error) {
  console.error('Hyperspell error:', error);
  // Return a user-friendly error or fallback
  throw new Error('Failed to search memories');
}
```

## Framework Detection Hints

If auto-detection didn't work, look for these indicators:

| Framework | Detection Signals |
|-----------|-------------------|
| Next.js App Router | `app/` directory, `next.config.js` |
| Next.js Pages Router | `pages/` directory, `next.config.js` |
| Express | `express` in package.json, `app.use()` patterns |
| FastAPI | `fastapi` in requirements, `@app.get()` decorators |
| React SPA | `react` without `next`, `vite.config.js` or CRA setup |
