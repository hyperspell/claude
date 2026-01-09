# OAuth Account Connection

Let your users connect their accounts (Gmail, Slack, etc.) so Hyperspell can automatically ingest their data and create searchable memories.

## Overview

The OAuth flow works as follows:
1. Your backend generates a user token using your API key
2. Your frontend redirects to `connect.hyperspell.com` with that token
3. User authorizes their accounts on the Hyperspell Connect page
4. User is redirected back to your app
5. Hyperspell automatically ingests data from connected accounts

## Step 1: Get Available Integrations

First, fetch the available integrations for your project:

```sh
curl --request GET \
  --url https://api.hyperspell.com/integrations/list \
  --header 'Authorization: Bearer YOUR_API_KEY'
```

Note the `provider` names returned (e.g., `gmail`, `slack`, `google_drive`). You'll use these when displaying which services users can connect.

## Step 2: Implement Token Generation

Your backend needs to generate user tokens. The implementation depends on your framework:

- **Next.js App Router** → See `../frameworks/nextjs_app_router.md`
- **Next.js Pages Router** → See `../frameworks/nextjs_pages_router.md`
- **Express.js** → See `../frameworks/express.md`
- **FastAPI** → See `../frameworks/fastapi.md`
- **Other frameworks** → See `../frameworks/SKILL.md` for adaptation guidance

The basic pattern is:

```typescript
// Server-side only
import Hyperspell from 'hyperspell';

async function generateUserToken(userId: string): Promise<string> {
  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
  });

  const response = await hyperspell.auth.userToken({ user_id: userId });
  return response.token;
}
```

## Step 3: Implement Memory Search

Also create a search function for querying memories:

```typescript
// Server-side only
import Hyperspell from 'hyperspell';

async function searchMemories(userId: string, query: string): Promise<string> {
  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await hyperspell.memories.search({
    query,
    answer: true,
    sources: ['gmail', 'slack'], // Use providers from Step 1
  });

  return response.answer;
}
```

## Step 4: Add Connect Button

Add a button to your UI that:
1. Fetches a user token from your backend
2. Redirects to Hyperspell Connect with the token

### Button Placement

Analyze the codebase to find an appropriate location:
- Settings page or modal
- User profile/account section
- Onboarding flow
- Sidebar or navigation menu

Ask the user where they'd like to place the button.

### Hyperspell Logo

Use this SVG for the button icon (adjust fill color as needed):

```xml
<svg width="32" height="32" viewBox="0 0 32 32" fill="none" xmlns="http://www.w3.org/2000/svg">
<path d="M27.963 25.1327C28.1219 25.409 28.0219 25.7361 27.7881 25.8964C27.6981 25.9581 27.5867 25.9958 27.4639 25.996H4.57721C4.13467 25.9958 3.85733 25.5161 4.07819 25.1327L5.30865 23.0009H26.7325L27.963 25.1327ZM25.4542 20.7831C25.4956 20.8557 25.4898 20.9379 25.4522 21.0009H15.8975C15.865 20.9474 15.854 20.8801 15.877 20.8143C16.0196 20.4085 16.1474 19.9952 16.2598 19.576C16.2961 19.4408 16.1927 19.3089 16.0528 19.3085H7.81061C7.64482 19.3083 7.54145 19.1289 7.62408 18.9852L8.76862 16.9999H23.2725L25.4542 20.7831ZM19.794 11.1835C19.8706 11.1838 19.9422 11.2246 19.9805 11.2909L22.0118 14.8143C22.0466 14.8752 22.0469 14.9424 22.0245 14.9999H16.8467C16.8438 14.5065 16.82 14.0176 16.7764 13.535C16.7666 13.4251 16.6739 13.3411 16.5635 13.3407H11.252C11.0865 13.3405 10.9832 13.1612 11.0655 13.0175L12.0606 11.2909C12.0991 11.2244 12.1704 11.1836 12.2471 11.1835H19.794ZM15.5225 5.28894C15.7438 4.90519 16.2974 4.90516 16.5186 5.28894L18.6592 8.99988H15.7305C15.4488 8.25709 15.1204 7.53696 14.7432 6.84753C14.7072 6.78125 14.7086 6.70104 14.7462 6.63562L15.5225 5.28894Z" fill="#000000"/>
</svg>
```

### Connect URL Format

The connect URL follows this pattern:

```
https://connect.hyperspell.com?token={userToken}&redirect_uri={currentPageUrl}
```

- `token`: The user token from your backend
- `redirect_uri`: Where to send the user after connecting (usually the current page)

## Flow-Specific Instructions

### FLOW A: Monorepo/Fullstack

1. Backend: Create token endpoint (see framework guides)
2. Frontend: Call backend to get token, then redirect to connect.hyperspell.com
3. Follow the appropriate framework guide for implementation

### FLOW B: Backend-only

1. Backend: Create token endpoint
2. Backend: Either serve a simple HTML page with the connect button, or create an API endpoint that returns a redirect URL
3. Users access the connect flow via a URL you provide

Example redirect endpoint:

```typescript
app.get('/connect', async (req, res) => {
  const userId = req.user.id;
  const token = await generateUserToken(userId);
  const redirectUri = `${req.protocol}://${req.get('host')}/connect/callback`;
  res.redirect(`https://connect.hyperspell.com?token=${token}&redirect_uri=${redirectUri}`);
});
```

## After Connection

Once users connect their accounts:
- Hyperspell automatically starts ingesting their data
- Use the `/memories/status` endpoint to check indexing progress
- Data becomes searchable via `/memories/query` as it's indexed

Display this message to the user after implementation:

```
Great! I've set up the Hyperspell Connect flow. Your users can now connect their accounts, and Hyperspell will automatically create searchable memories from their data.

Feel free to test it out!
```
