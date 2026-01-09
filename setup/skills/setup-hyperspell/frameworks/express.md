# Express.js Integration

This guide covers integrating Hyperspell with Express.js backends.

## Setup Routes

Create a router for Hyperspell endpoints:

```typescript
// routes/hyperspell.ts
import { Router, Request, Response } from 'express';
import Hyperspell from 'hyperspell';

const router = Router();

// Middleware to get user ID - adapt to your auth system
function getUserId(req: Request): string {
  // Passport.js example:
  // return req.user?.id || 'anonymous';

  // JWT example:
  // return req.auth?.userId || 'anonymous';

  // Session example:
  // return req.session?.userId || 'anonymous';

  // TODO: Replace with your actual auth implementation
  return 'anonymous';
}

// POST /api/hyperspell/token - Generate user token
router.post('/token', async (req: Request, res: Response) => {
  const userId = getUserId(req);

  if (!userId || userId === 'anonymous') {
    // Decide if you want to allow anonymous users
    // return res.status(401).json({ error: 'Unauthorized' });
  }

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

// POST /api/hyperspell/search - Search memories
router.post('/search', async (req: Request, res: Response) => {
  const userId = getUserId(req);
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

    res.json({ answer: response.answer });
  } catch (error) {
    console.error('Failed to search:', error);
    res.status(500).json({ error: 'Failed to search memories' });
  }
});

// GET /api/hyperspell/connect - Redirect to Hyperspell Connect (FLOW B)
router.get('/connect', async (req: Request, res: Response) => {
  const userId = getUserId(req);

  try {
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
    });

    const response = await hyperspell.auth.userToken({ user_id: userId });
    const redirectUri = `${req.protocol}://${req.get('host')}/api/hyperspell/callback`;
    const connectUrl = `https://connect.hyperspell.com?token=${response.token}&redirect_uri=${encodeURIComponent(redirectUri)}`;

    res.redirect(connectUrl);
  } catch (error) {
    console.error('Failed to redirect to connect:', error);
    res.status(500).json({ error: 'Failed to initiate connection' });
  }
});

// GET /api/hyperspell/callback - Handle redirect after connection
router.get('/callback', (req: Request, res: Response) => {
  // User has connected their accounts
  // Redirect to your app's success page
  res.redirect('/settings?connected=true');
});

export default router;
```

## Register the Router

Add the router to your Express app:

```typescript
// app.ts or server.ts
import express from 'express';
import hyperspellRouter from './routes/hyperspell';

const app = express();

app.use(express.json());

// Your auth middleware
// app.use(passport.initialize());
// app.use(passport.session());

// Mount Hyperspell routes
app.use('/api/hyperspell', hyperspellRouter);

// ... rest of your app
```

## Direct Memory Operations

For FLOW C (backend with direct memory management), create additional routes:

```typescript
// routes/hyperspell.ts (add to existing router)
import multer from 'multer';

const upload = multer({ storage: multer.memoryStorage() });

// POST /api/hyperspell/memories/add - Add a memory
router.post('/memories/add', async (req: Request, res: Response) => {
  const userId = getUserId(req);
  const { content, title, metadata } = req.body;

  if (!content) {
    return res.status(400).json({ error: 'Content is required' });
  }

  try {
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
      userID: userId,
    });

    const response = await hyperspell.memories.add({
      content,
      title,
      source: 'custom',
      metadata,
    });

    res.json(response);
  } catch (error) {
    console.error('Failed to add memory:', error);
    res.status(500).json({ error: 'Failed to add memory' });
  }
});

// POST /api/hyperspell/memories/upload - Upload a file
router.post('/memories/upload', upload.single('file'), async (req: Request, res: Response) => {
  const userId = getUserId(req);

  if (!req.file) {
    return res.status(400).json({ error: 'File is required' });
  }

  try {
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
      userID: userId,
    });

    const response = await hyperspell.memories.upload({
      file: req.file.buffer,
      filename: req.file.originalname,
    });

    res.json(response);
  } catch (error) {
    console.error('Failed to upload file:', error);
    res.status(500).json({ error: 'Failed to upload file' });
  }
});

// GET /api/hyperspell/memories - List memories
router.get('/memories', async (req: Request, res: Response) => {
  const userId = getUserId(req);
  const { limit = '20', offset = '0', source } = req.query;

  try {
    const hyperspell = new Hyperspell({
      apiKey: process.env.HYPERSPELL_API_KEY!,
      userID: userId,
    });

    const response = await hyperspell.memories.list({
      limit: parseInt(limit as string),
      offset: parseInt(offset as string),
      source: source as string | undefined,
    });

    res.json(response);
  } catch (error) {
    console.error('Failed to list memories:', error);
    res.status(500).json({ error: 'Failed to list memories' });
  }
});
```

## Environment Variables

Make sure to set in your `.env`:

```
HYPERSPELL_API_KEY=hs-0-your-api-key
```

## Auth Middleware Examples

### Passport.js with JWT

```typescript
import passport from 'passport';
import { Strategy as JwtStrategy, ExtractJwt } from 'passport-jwt';

passport.use(new JwtStrategy({
  jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
  secretOrKey: process.env.JWT_SECRET,
}, (payload, done) => {
  // Find user by payload.sub
  done(null, { id: payload.sub });
}));

// In routes:
router.post('/token', passport.authenticate('jwt', { session: false }), async (req, res) => {
  const userId = req.user.id;
  // ...
});
```

### Express Session

```typescript
import session from 'express-session';

app.use(session({
  secret: process.env.SESSION_SECRET!,
  resave: false,
  saveUninitialized: false,
}));

// In routes:
function getUserId(req: Request): string {
  return req.session?.userId || 'anonymous';
}
```

## Frontend Integration

If you have a separate frontend, it can call these endpoints:

```typescript
// Frontend code
async function getHyperspellToken(): Promise<string> {
  const response = await fetch('/api/hyperspell/token', {
    method: 'POST',
    credentials: 'include', // Include cookies for session auth
  });
  const { token } = await response.json();
  return token;
}

async function connectHyperspell() {
  const token = await getHyperspellToken();
  const redirectUri = window.location.href;
  window.location.href = `https://connect.hyperspell.com?token=${token}&redirect_uri=${redirectUri}`;
}
```
