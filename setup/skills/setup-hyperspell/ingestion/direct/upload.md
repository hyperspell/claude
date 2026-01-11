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

## Common Use Cases

### Upload from existing file upload handler

If your app already has a file upload endpoint, call the helper from there:

```typescript
// In your existing upload handler
const result = await uploadFile({
  userId: currentUser.id,
  filePath: savedFilePath,
  collection: 'documents',
});
```

### Process files from a directory

```typescript
import fs from 'fs';
import path from 'path';

async function processDocumentsFolder(userId: string, folderPath: string) {
  const files = fs.readdirSync(folderPath);

  for (const filename of files) {
    await uploadFile({
      userId,
      filePath: path.join(folderPath, filename),
      collection: 'imported_docs',
    });
  }
}
```

### Upload with metadata

```typescript
await uploadFile({
  userId: currentUser.id,
  filePath: '/path/to/report.pdf',
  collection: 'reports',
  metadata: {
    department: 'engineering',
    year: 2024,
  },
});
```

## Integration Notes

- Call this helper from your existing code (e.g., after saving a user-uploaded file, during a batch import job)
- Files are processed asynchronously - they may not be immediately searchable
- The `status` field indicates processing state: `pending`, `processing`, `completed`, or `failed`
- Use the returned `resource_id` to track or reference the uploaded file
