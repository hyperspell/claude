# Frontend-Only Integration

This guide is for frontend-only applications (SPAs) that don't have their own backend. Since you can't safely store the Hyperspell API key in frontend code, you need an external source for user tokens.

## The Challenge

- API keys **cannot** be exposed in frontend code (security risk)
- Hyperspell operations require either an API key or a user token
- User tokens must be generated server-side using your API key

## Solutions

Ask the user which situation applies:

```
Your project appears to be frontend-only. To use Hyperspell securely, you need a way to get user tokens. Which option works for you?

1. I have a backend endpoint that can provide tokens
2. I use Clerk or Auth0 for authentication
3. I'll set this up later (generate placeholder code)
```

### Option 1: Existing Backend Endpoint

If the user has a separate backend service:

Ask for the endpoint URL:
```
What's the URL of your backend endpoint that will provide Hyperspell user tokens?
(e.g., https://api.yourapp.com/hyperspell/token)
```

Generate a Hyperspell client wrapper:

```typescript
// lib/hyperspell.ts

let cachedToken: string | null = null;
let tokenExpiry: number = 0;

async function getHyperspellToken(): Promise<string> {
  // Return cached token if still valid (with 5min buffer)
  if (cachedToken && Date.now() < tokenExpiry - 300000) {
    return cachedToken;
  }

  // Fetch new token from your backend
  const response = await fetch('YOUR_BACKEND_ENDPOINT', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      // Include your auth headers
      'Authorization': `Bearer ${getAuthToken()}`,
    },
  });

  if (!response.ok) {
    throw new Error('Failed to get Hyperspell token');
  }

  const data = await response.json();
  cachedToken = data.token;
  // Assume 1 hour expiry, adjust as needed
  tokenExpiry = Date.now() + 3600000;

  return cachedToken;
}

export async function searchMemories(query: string) {
  const token = await getHyperspellToken();

  const response = await fetch('https://api.hyperspell.com/memories/query', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ query, answer: true }),
  });

  return response.json();
}

export async function getConnectUrl(redirectUri: string): Promise<string> {
  const token = await getHyperspellToken();
  return `https://connect.hyperspell.com?token=${token}&redirect_uri=${encodeURIComponent(redirectUri)}`;
}
```

Replace `YOUR_BACKEND_ENDPOINT` with the actual endpoint URL.
Replace `getAuthToken()` with your auth system's token getter.

### Option 2: Clerk Integration

If using Clerk, you can configure Clerk to provide Hyperspell tokens via JWT templates:

1. **In Clerk Dashboard:**
   - Go to JWT Templates
   - Create a new template for Hyperspell
   - Configure it to include the user ID claim

2. **Contact Hyperspell** to set up Clerk as a token provider for your project.

3. **In your frontend:**

```typescript
import { useAuth } from '@clerk/nextjs';

export function useHyperspell() {
  const { getToken } = useAuth();

  async function getHyperspellToken() {
    // Get a token with the Hyperspell template
    const token = await getToken({ template: 'hyperspell' });
    return token;
  }

  async function searchMemories(query: string) {
    const token = await getHyperspellToken();

    const response = await fetch('https://api.hyperspell.com/memories/query', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ query, answer: true }),
    });

    return response.json();
  }

  return { searchMemories, getHyperspellToken };
}
```

### Option 3: Auth0 Integration

Similar to Clerk, Auth0 can be configured to issue Hyperspell-compatible tokens:

1. **In Auth0 Dashboard:**
   - Create an API for Hyperspell
   - Add a custom action to include necessary claims

2. **Contact Hyperspell** to set up Auth0 as a token provider.

3. **In your frontend:**

```typescript
import { useAuth0 } from '@auth0/auth0-react';

export function useHyperspell() {
  const { getAccessTokenSilently } = useAuth0();

  async function getHyperspellToken() {
    const token = await getAccessTokenSilently({
      audience: 'https://api.hyperspell.com',
    });
    return token;
  }

  async function searchMemories(query: string) {
    const token = await getHyperspellToken();

    const response = await fetch('https://api.hyperspell.com/memories/query', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ query, answer: true }),
    });

    return response.json();
  }

  return { searchMemories, getHyperspellToken };
}
```

### Option 4: Set Up Later (TODO Placeholder)

Generate placeholder code with TODOs:

```typescript
// lib/hyperspell.ts
// TODO: Set up a backend endpoint to provide Hyperspell user tokens
// Your backend should call POST https://api.hyperspell.com/auth/user_token
// with your API key and return the token to this frontend.
//
// You can use the /setup:hype skill on your backend project to set this up.

async function getHyperspellToken(): Promise<string> {
  // TODO: Replace with your actual token endpoint
  throw new Error(
    'Hyperspell token endpoint not configured. ' +
    'Set up a backend endpoint that calls /auth/user_token with your API key.'
  );
}

export async function searchMemories(query: string) {
  const token = await getHyperspellToken();

  const response = await fetch('https://api.hyperspell.com/memories/query', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ query, answer: true }),
  });

  return response.json();
}

export async function getConnectUrl(redirectUri: string): Promise<string> {
  const token = await getHyperspellToken();
  return `https://connect.hyperspell.com?token=${token}&redirect_uri=${encodeURIComponent(redirectUri)}`;
}
```

Display this message to the user:

```
I've created placeholder code for Hyperspell integration. Before it will work, you'll need to:

1. Set up a backend endpoint that generates Hyperspell user tokens
2. Update the getHyperspellToken() function with your endpoint URL

You can use the /setup:hype skill on your backend project to create the token endpoint, then come back and update this code.
```

## Connect Button for Frontend-Only

Once tokens are working, add a connect button:

```tsx
import { useState } from 'react';
import { getConnectUrl } from '@/lib/hyperspell';

export function HyperspellConnectButton() {
  const [loading, setLoading] = useState(false);

  async function handleConnect() {
    setLoading(true);
    try {
      const connectUrl = await getConnectUrl(window.location.href);
      window.location.href = connectUrl;
    } catch (error) {
      console.error('Failed to get connect URL:', error);
      alert('Failed to connect. Please try again.');
    } finally {
      setLoading(false);
    }
  }

  return (
    <button onClick={handleConnect} disabled={loading}>
      {loading ? 'Connecting...' : 'Connect Your Accounts'}
    </button>
  );
}
```
