# Upload Files

Upload files to Hyperspell for automatic processing and indexing. Hyperspell will extract text from various file formats and create searchable memories.

## API Reference

**Endpoint:** `POST /memories/upload`

**Headers:**
- `Authorization: Bearer <API_KEY>` or `Bearer <USER_TOKEN>`
- `Content-Type: multipart/form-data`
- `X-As-User: <user_id>` (only when using API key, not with user token)

**Request Body:** `multipart/form-data` with file field

## Supported File Types

Hyperspell can process:
- **Documents**: PDF, DOCX, DOC, TXT, MD
- **Spreadsheets**: XLSX, XLS, CSV
- **Presentations**: PPTX, PPT
- **Other**: HTML, JSON

## TypeScript Implementation

Create a file upload utility:

```typescript
import Hyperspell from 'hyperspell';

interface UploadFileParams {
  userId: string;
  file: File | Buffer;
  filename: string;
  metadata?: Record<string, unknown>;
}

export async function uploadFile({ userId, file, filename, metadata }: UploadFileParams) {
  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await client.memories.upload({
    file,
    filename,
    metadata,
  });

  return response;
}
```

## Python Implementation

```python
import os
from hyperspell import Hyperspell

def upload_file(user_id: str, file_path: str, metadata: dict = None):
    """Upload a file for processing and indexing."""
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    with open(file_path, 'rb') as f:
        response = client.memories.upload(
            file=f,
            filename=os.path.basename(file_path),
            metadata=metadata or {}
        )

    return response
```

## Raw API (No SDK)

For direct HTTP uploads:

```typescript
async function uploadFile(userId: string, file: File) {
  const formData = new FormData();
  formData.append('file', file);

  const response = await fetch('https://api.hyperspell.com/memories/upload', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.HYPERSPELL_API_KEY}`,
      'X-As-User': userId,
    },
    body: formData,
  });

  if (!response.ok) {
    throw new Error(`Failed to upload file: ${response.statusText}`);
  }

  return response.json();
}
```

## Framework-Specific Examples

### Next.js API Route (App Router)

```typescript
// app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { uploadFile } from '@/lib/hyperspell';
import { auth } from '@/lib/auth'; // Your auth solution

export async function POST(request: NextRequest) {
  const session = await auth();
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const formData = await request.formData();
  const file = formData.get('file') as File;

  if (!file) {
    return NextResponse.json({ error: 'No file provided' }, { status: 400 });
  }

  const result = await uploadFile({
    userId: session.user.id,
    file,
    filename: file.name,
  });

  return NextResponse.json(result);
}
```

### Express.js with Multer

```typescript
import express from 'express';
import multer from 'multer';
import { uploadFile } from './hyperspell';

const upload = multer({ storage: multer.memoryStorage() });

app.post('/api/upload', upload.single('file'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file provided' });
  }

  const result = await uploadFile({
    userId: req.user.id, // From your auth middleware
    file: req.file.buffer,
    filename: req.file.originalname,
  });

  res.json(result);
});
```

### FastAPI

```python
from fastapi import FastAPI, UploadFile, Depends
from hyperspell import Hyperspell
import os

app = FastAPI()

@app.post("/api/upload")
async def upload_document(
    file: UploadFile,
    user_id: str = Depends(get_current_user_id)  # Your auth dependency
):
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    contents = await file.read()
    response = client.memories.upload(
        file=contents,
        filename=file.filename
    )

    return response
```

## Processing Status

File uploads are processed asynchronously. To check processing status:

```typescript
// Check indexing status
const status = await client.memories.status();
console.log(status);
// { total: 100, indexed: 95, pending: 5 }
```

## Integration Notes

- Files are processed in the background - they may not be immediately searchable
- Use the status endpoint to monitor indexing progress
- Set appropriate file size limits in your upload endpoints
- Consider adding progress indicators for large file uploads
- Store the returned memory ID if you need to reference or delete the file later
