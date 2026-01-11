# Upload Files

Upload files to Hyperspell for automatic processing and indexing. Hyperspell will extract text from various file formats and create searchable memories.

## API Reference

**Endpoint:** `POST /memories/upload`

**Headers:**
- `Authorization: Bearer <API_KEY>` or `Bearer <USER_TOKEN>`
- `Content-Type: multipart/form-data`
- `X-As-User: <user_id>` (only when using API key, not with user token)

**Request Body:** `multipart/form-data`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | file | Yes | The file to ingest |
| `collection` | string | No | Collection to organize the document |
| `metadata` | string (JSON) | No | Custom metadata for filtering (alphanumeric keys, max 64 chars) |

**Response:**
```json
{
  "source": "collections",
  "resource_id": "<string>",
  "status": "pending"
}
```

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
import fs from 'fs';

interface UploadFileParams {
  userId: string;
  filePath: string;
  collection?: string;
  metadata?: Record<string, string | number | boolean>;
}

export async function uploadFile({ userId, filePath, collection, metadata }: UploadFileParams) {
  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const response = await client.memories.upload({
    file: fs.createReadStream(filePath),
    collection,
    metadata,
  });

  return response;
}
```

## Python Implementation

```python
import os
from hyperspell import Hyperspell

def upload_file(user_id: str, file_path: str, collection: str = None, metadata: dict = None):
    """Upload a file for processing and indexing."""
    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    with open(file_path, 'rb') as f:
        response = client.memories.upload(
            file=f.read(),
            collection=collection,
            metadata=metadata
        )

    return response
```

## Framework-Specific Examples

### Next.js API Route (App Router)

```typescript
// app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Hyperspell from 'hyperspell';

export async function POST(request: NextRequest) {
  const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules

  const formData = await request.formData();
  const file = formData.get('file') as File;

  if (!file) {
    return NextResponse.json({ error: 'No file provided' }, { status: 400 });
  }

  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const result = await client.memories.upload({
    file: Buffer.from(await file.arrayBuffer()),
  });

  return NextResponse.json(result);
}
```

### Express.js with Multer

```typescript
import express from 'express';
import multer from 'multer';
import Hyperspell from 'hyperspell';

const upload = multer({ storage: multer.memoryStorage() });

app.post('/api/upload', upload.single('file'), async (req, res) => {
  const userId = ... // Write code to get the ID of the currently logged in user here — you might have to import other modules

  if (!req.file) {
    return res.status(400).json({ error: 'No file provided' });
  }

  const client = new Hyperspell({
    apiKey: process.env.HYPERSPELL_API_KEY!,
    userID: userId,
  });

  const result = await client.memories.upload({
    file: req.file.buffer,
  });

  res.json(result);
});
```

### FastAPI

```python
from fastapi import FastAPI, UploadFile
from hyperspell import Hyperspell
import os

app = FastAPI()

@app.post("/api/upload")
async def upload_document(file: UploadFile):
    user_id = ...  # Write code to get the ID of the currently logged in user here — you might have to import other modules

    client = Hyperspell(
        api_key=os.environ["HYPERSPELL_API_KEY"],
        user_id=user_id
    )

    contents = await file.read()
    response = client.memories.upload(file=contents)

    return response
```

## Integration Notes

- Files are processed asynchronously - they may not be immediately searchable
- The `status` field indicates processing state: `pending`, `processing`, `completed`, or `failed`
- Use the returned `resource_id` to track or reference the uploaded file
- Set appropriate file size limits in your upload endpoints
- Consider adding progress indicators for large file uploads
