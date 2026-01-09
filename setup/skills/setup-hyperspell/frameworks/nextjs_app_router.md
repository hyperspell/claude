# Next.js App Router Integration

This guide covers integrating Hyperspell with Next.js projects using the App Router (the `app/` directory).

## Server Actions Setup

Create a server actions file for Hyperspell operations:

```typescript
// app/actions/hyperspell.ts (or src/actions/hyperspell.ts)
'use server'

import Hyperspell from 'hyperspell';

// Helper to get current user ID - adapt to your auth system
async function getCurrentUserId(): Promise<string> {
  // Examples for common auth systems:

  // NextAuth.js
  // import { getServerSession } from 'next-auth';
  // import { authOptions } from '@/lib/auth';
  // const session = await getServerSession(authOptions);
  // return session?.user?.id || 'anonymous';

  // Clerk
  // import { auth } from '@clerk/nextjs';
  // const { userId } = auth();
  // return userId || 'anonymous';

  // TODO: Replace with your actual auth implementation
  return 'anonymous';
}

export async function getUserToken(): Promise<string> {
  const userId = await getCurrentUserId();

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
  });

  const response = await hyperspell.auth.userToken({ user_id: userId });
  return response.token;
}

export async function searchMemories(query: string, answer: boolean = true) {
  const userId = await getCurrentUserId();

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await hyperspell.memories.search({
    query,
    answer,
    sources: ['gmail', 'slack'], // Replace with your providers from /integrations/list
  });

  return response.answer;
}
```

Adapt the `getCurrentUserId()` function to your authentication system.

## Connect Button Component

Create a client component for the connect button:

```tsx
// components/HyperspellConnectButton.tsx
'use client'

import { useState } from 'react';
import { getUserToken } from '@/app/actions/hyperspell';

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

export function HyperspellConnectButton() {
  const [loading, setLoading] = useState(false);

  async function handleConnect() {
    setLoading(true);
    try {
      const token = await getUserToken();
      const redirectUri = window.location.href;
      const connectUrl = `https://connect.hyperspell.com?token=${token}&redirect_uri=${encodeURIComponent(redirectUri)}`;
      window.location.href = connectUrl;
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

Adapt the button styling to match your project's design system.

## Button Placement

Add the button component where users can find it. Common locations:

### In a Settings Page

```tsx
// app/settings/page.tsx
import { HyperspellConnectButton } from '@/components/HyperspellConnectButton';

export default function SettingsPage() {
  return (
    <div>
      <h1>Settings</h1>

      <section>
        <h2>Connected Accounts</h2>
        <p>Connect your accounts to enable AI-powered search across your data.</p>
        <HyperspellConnectButton />
      </section>
    </div>
  );
}
```

### In a Sidebar or Navigation

```tsx
// components/Sidebar.tsx
import { HyperspellConnectButton } from '@/components/HyperspellConnectButton';

export function Sidebar() {
  return (
    <nav>
      {/* Other navigation items */}
      <div className="mt-auto p-4">
        <HyperspellConnectButton />
      </div>
    </nav>
  );
}
```

## API Route Alternative

If you prefer API routes over server actions:

```typescript
// app/api/hyperspell/token/route.ts
import { NextResponse } from 'next/server';
import Hyperspell from 'hyperspell';
import { auth } from '@/lib/auth'; // Your auth

export async function POST() {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
  });

  const response = await hyperspell.auth.userToken({
    user_id: session.user.id,
  });

  return NextResponse.json({ token: response.token });
}
```

Then update the button to use the API route:

```typescript
async function handleConnect() {
  const response = await fetch('/api/hyperspell/token', { method: 'POST' });
  const { token } = await response.json();
  // ... rest of redirect logic
}
```

## FLOW C: Direct Memory Management (No OAuth)

If using direct memory management without OAuth, you don't need the connect button. Just use the server actions for memory operations:

```typescript
// In your server components or actions
import Hyperspell from 'hyperspell';

export async function addMemory(content: string, title?: string) {
  const userId = await getCurrentUserId();

  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  return hyperspell.memories.add({
    content,
    title,
    source: 'custom',
  });
}
```
