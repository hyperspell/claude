# Next.js Pages Router Integration

This guide covers integrating Hyperspell with Next.js projects using the Pages Router (the `pages/` directory).

## API Routes Setup

Create API routes for Hyperspell operations:

### Token Endpoint

```typescript
// pages/api/hyperspell/token.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import Hyperspell from 'hyperspell';
// Import your auth - examples below

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  // Get user ID from your auth system
  // NextAuth.js example:
  // import { getServerSession } from 'next-auth';
  // import { authOptions } from '@/lib/auth';
  // const session = await getServerSession(req, res, authOptions);
  // const userId = session?.user?.id;

  // TODO: Replace with your actual auth implementation
  const userId = 'anonymous';

  if (!userId) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  try {
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
    });

    const response = await hyperspell.auth.userToken({ user_id: userId });
    return res.status(200).json({ token: response.token });
  } catch (error) {
    console.error('Failed to generate token:', error);
    return res.status(500).json({ error: 'Failed to generate token' });
  }
}
```

### Search Endpoint

```typescript
// pages/api/hyperspell/search.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import Hyperspell from 'hyperspell';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  // TODO: Get user ID from your auth system
  const userId = 'anonymous';

  if (!userId) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const { query } = req.body;

  if (!query) {
    return res.status(400).json({ error: 'Query is required' });
  }

  try {
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
      userID: userId,
    });

    const response = await hyperspell.memories.search({
      query,
      answer: true,
      sources: ['gmail', 'slack'], // Replace with your providers
    });

    return res.status(200).json({ answer: response.answer });
  } catch (error) {
    console.error('Failed to search:', error);
    return res.status(500).json({ error: 'Failed to search memories' });
  }
}
```

## Connect Button Component

Create a React component for the connect button:

```tsx
// components/HyperspellConnectButton.tsx
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

export function HyperspellConnectButton() {
  const [loading, setLoading] = useState(false);

  async function handleConnect() {
    setLoading(true);
    try {
      const response = await fetch('/api/hyperspell/token', {
        method: 'POST',
      });

      if (!response.ok) {
        throw new Error('Failed to get token');
      }

      const { token } = await response.json();
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
      className="inline-flex items-center gap-2 px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 disabled:opacity-50"
    >
      <HyperspellLogo />
      {loading ? 'Connecting...' : 'Connect Your Accounts'}
    </button>
  );
}
```

## Usage in Pages

### Settings Page Example

```tsx
// pages/settings.tsx
import { HyperspellConnectButton } from '@/components/HyperspellConnectButton';

export default function SettingsPage() {
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-6">Settings</h1>

      <section className="mb-8">
        <h2 className="text-xl font-semibold mb-2">Connected Accounts</h2>
        <p className="text-gray-600 mb-4">
          Connect your accounts to enable AI-powered search across your data.
        </p>
        <HyperspellConnectButton />
      </section>
    </div>
  );
}
```

### With getServerSideProps

If you need to check connection status server-side:

```tsx
// pages/settings.tsx
import { GetServerSideProps } from 'next';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';
import Hyperspell from 'hyperspell';
import { HyperspellConnectButton } from '@/components/HyperspellConnectButton';

interface Props {
  isConnected: boolean;
  providers: string[];
}

export const getServerSideProps: GetServerSideProps<Props> = async (context) => {
  const session = await getServerSession(context.req, context.res, authOptions);

  if (!session?.user?.id) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    };
  }

  // Check if user has any connected integrations
  const hyperspell = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: session.user.id,
  });

  try {
    const status = await hyperspell.memories.status();
    return {
      props: {
        isConnected: status.total > 0,
        providers: ['gmail', 'slack'], // From /integrations/list
      },
    };
  } catch {
    return {
      props: {
        isConnected: false,
        providers: [],
      },
    };
  }
};

export default function SettingsPage({ isConnected, providers }: Props) {
  return (
    <div>
      <h1>Settings</h1>
      {isConnected ? (
        <p>Your accounts are connected!</p>
      ) : (
        <>
          <p>Connect your {providers.join(', ')} accounts:</p>
          <HyperspellConnectButton />
        </>
      )}
    </div>
  );
}
```

## Using the Search API

Create a hook for searching memories:

```typescript
// hooks/useHyperspellSearch.ts
import { useState } from 'react';

export function useHyperspellSearch() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  async function search(query: string): Promise<string | null> {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/hyperspell/search', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ query }),
      });

      if (!response.ok) {
        throw new Error('Search failed');
      }

      const { answer } = await response.json();
      return answer;
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Search failed');
      return null;
    } finally {
      setLoading(false);
    }
  }

  return { search, loading, error };
}
```
