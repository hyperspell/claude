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

## Step 2: Create Token Endpoint

Your backend needs an endpoint that generates user tokens. This is the **only** Hyperspell-specific endpoint you need to create.

The endpoint should:
1. Authenticate the current user (using your auth system)
2. Call Hyperspell's `/auth/user_token` with your API key and the user's ID
3. Return the token to the frontend

### Next.js App Router (Server Action)

```typescript
// app/actions/hyperspell.ts
'use server'

import Hyperspell from 'hyperspell';

export async function getHyperspellToken(): Promise<string> {
  const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules. IMPORTANT: If no user auth logic can be found, just create a dummy ID here and add a TODO.

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
  });

  const response = await hyperspell.auth.userToken({
    user_id: session.user.id,
  });

  return response.token;
}
```

### Next.js App Router (API Route)

```typescript
// app/api/hyperspell/token/route.ts
import { NextResponse } from 'next/server';
import Hyperspell from 'hyperspell';

export async function POST() {
  const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules. IMPORTANT: If no user auth logic can be found, just create a dummy ID here and add a TODO.

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
  });

  const response = await hyperspell.auth.userToken({
    user_id: userId,
  });

  return NextResponse.json({ token: response.token });
}
```

### Next.js Pages Router

```typescript
// pages/api/hyperspell/token.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import Hyperspell from 'hyperspell';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules. IMPORTANT: If no user auth logic can be found, just create a dummy ID here and add a TODO.

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
  });

  const response = await hyperspell.auth.userToken({
    user_id: userId,
  });

  res.json({ token: response.token });
}
```

### Express.js

```typescript
import { Router } from 'express';
import Hyperspell from 'hyperspell';

const router = Router();

router.post('/hyperspell/token', async (req, res) => {
  const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules. IMPORTANT: If no user auth logic can be found, just create a dummy ID here and add a TODO.

  try {
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
    });

    const response = await hyperspell.auth.userToken({ user_id: userId });
    res.json({ token: response.token });
  } catch (error) {
    console.error('Failed to generate token:', error);
    res.status(500).json({ error: 'Failed to generate token' });
  }
});

export default router;
```

### FastAPI

```python
from fastapi import APIRouter, HTTPException
from hyperspell import Hyperspell
import os

router = APIRouter()

@router.post("/hyperspell/token")
async def get_hyperspell_token():
    user_id = ...  # Write code to get the ID of the currently logged in user here — you might have to import other modules. IMPORTANT: If no user auth logic can be found, just create a dummy ID here and add a TODO.

    hyperspell = Hyperspell(api_key=os.environ["HYPERSPELL_API_KEY"])
    response = hyperspell.auth.user_token(user_id=user_id)

    return {"token": response.token}
```

### Other Frameworks

The pattern is the same for any framework:
1. Get the authenticated user ID from your auth system
2. Call `hyperspell.auth.userToken({ user_id })` with your API key
3. Return the token

## Step 3: Add Connect Button

Add a button to your UI that fetches a token and redirects to Hyperspell Connect.

### React Component

```tsx
'use client'

import { useState } from 'react';

// Hyperspell logo SVG
function HyperspellLogo({ className }: { className?: string }) {
  return (
    <svg
      width="20"
      height="20"
      viewBox="0 0 32 32"
      fill="currentColor"
      className={className}
      xmlns="http://www.w3.org/2000/svg"
    >
      <path d="M27.963 25.1327C28.1219 25.409 28.0219 25.7361 27.7881 25.8964C27.6981 25.9581 27.5867 25.9958 27.4639 25.996H4.57721C4.13467 25.9958 3.85733 25.5161 4.07819 25.1327L5.30865 23.0009H26.7325L27.963 25.1327ZM25.4542 20.7831C25.4956 20.8557 25.4898 20.9379 25.4522 21.0009H15.8975C15.865 20.9474 15.854 20.8801 15.877 20.8143C16.0196 20.4085 16.1474 19.9952 16.2598 19.576C16.2961 19.4408 16.1927 19.3089 16.0528 19.3085H7.81061C7.64482 19.3083 7.54145 19.1289 7.62408 18.9852L8.76862 16.9999H23.2725L25.4542 20.7831ZM19.794 11.1835C19.8706 11.1838 19.9422 11.2246 19.9805 11.2909L22.0118 14.8143C22.0466 14.8752 22.0469 14.9424 22.0245 14.9999H16.8467C16.8438 14.5065 16.82 14.0176 16.7764 13.535C16.7666 13.4251 16.6739 13.3411 16.5635 13.3407H11.252C11.0865 13.3405 10.9832 13.1612 11.0655 13.0175L12.0606 11.2909C12.0991 11.2244 12.1704 11.1836 12.2471 11.1835H19.794ZM15.5225 5.28894C15.7438 4.90519 16.2974 4.90516 16.5186 5.28894L18.6592 8.99988H15.7305C15.4488 8.25709 15.1204 7.53696 14.7432 6.84753C14.7072 6.78125 14.7086 6.70104 14.7462 6.63562L15.5225 5.28894Z" />
    </svg>
  );
}

interface Props {
  getToken: () => Promise<string>;
}

export function HyperspellConnectButton({ getToken }: Props) {
  const [loading, setLoading] = useState(false);

  async function handleConnect() {
    setLoading(true);
    try {
      const token = await getToken();
      const redirectUri = window.location.href;
      window.location.href = `https://connect.hyperspell.com?token=${token}&redirect_uri=${encodeURIComponent(redirectUri)}`;
    } catch (error) {
      console.error('Failed to get Hyperspell token:', error);
      alert('Failed to connect. Please try again.');
      setLoading(false);
    }
  }

  return (
    <button
      onClick={handleConnect}
      disabled={loading}
      className="inline-flex items-center gap-2 px-4 py-2 bg-primary text-primary-foreground rounded-md hover:bg-primary/90 disabled:opacity-50"
    >
      <HyperspellLogo />
      {loading ? 'Connecting...' : 'Connect Your Accounts'}
    </button>
  );
}
```

### Usage with Server Action

```tsx
import { HyperspellConnectButton } from '@/components/HyperspellConnectButton';
import { getHyperspellToken } from '@/app/actions/hyperspell';

export function SettingsPage() {
  return (
    <div>
      <h2>Connected Accounts</h2>
      <HyperspellConnectButton getToken={getHyperspellToken} />
    </div>
  );
}
```

### Usage with API Route

```tsx
import { HyperspellConnectButton } from '@/components/HyperspellConnectButton';

async function getToken() {
  const response = await fetch('/api/hyperspell/token', { method: 'POST' });
  const { token } = await response.json();
  return token;
}

export function SettingsPage() {
  return <HyperspellConnectButton getToken={getToken} />;
}
```

### Button Placement (Required)

After creating the button component, you MUST ask the user where they want to place it. Use a multiple choice question with these options:
- Settings page
- User profile section
- Onboarding flow
- Sidebar
- Other (let user specify)

Once the user responds, add the button to that location. Adapt the button styling to match the project's design system.

## After Connection

Once users connect their accounts:
- Hyperspell automatically starts ingesting their data
- Data becomes searchable via the search API as it's indexed

Display this message after implementation:

```
Great! I've set up the Hyperspell Connect flow. Your users can now connect their accounts, and Hyperspell will automatically create searchable memories from their data.
```
