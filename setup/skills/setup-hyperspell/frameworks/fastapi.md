# FastAPI Integration

This guide covers integrating Hyperspell with FastAPI backends.

## Setup Router

Create a router for Hyperspell endpoints:

```python
# routers/hyperspell.py
import os
from fastapi import APIRouter, Depends, HTTPException, UploadFile, File
from pydantic import BaseModel
from hyperspell import Hyperspell
from typing import Optional

router = APIRouter(prefix="/api/hyperspell", tags=["hyperspell"])

# Dependency to get current user ID - adapt to your auth system
async def get_current_user_id() -> str:
    # FastAPI Users example:
    # async def get_current_user_id(user: User = Depends(current_active_user)) -> str:
    #     return str(user.id)

    # JWT example:
    # async def get_current_user_id(token: str = Depends(oauth2_scheme)) -> str:
    #     payload = decode_token(token)
    #     return payload.get("sub")

    # TODO: Replace with your actual auth implementation
    return "anonymous"


# Request/Response models
class TokenResponse(BaseModel):
    token: str


class SearchRequest(BaseModel):
    query: str


class SearchResponse(BaseModel):
    answer: str


class AddMemoryRequest(BaseModel):
    content: str
    title: Optional[str] = None
    metadata: Optional[dict] = None


class ListMemoriesResponse(BaseModel):
    memories: list
    total: int
    limit: int
    offset: int


# POST /api/hyperspell/token - Generate user token
@router.post("/token", response_model=TokenResponse)
async def get_token(user_id: str = Depends(get_current_user_id)):
    try:
        client = Hyperspell(api_key=os.environ["HYPERSPELL_API_KEY"])
        response = client.auth.user_token(user_id=user_id)
        return TokenResponse(token=response.token)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to generate token: {str(e)}")


# POST /api/hyperspell/search - Search memories
@router.post("/search", response_model=SearchResponse)
async def search_memories(
    request: SearchRequest,
    user_id: str = Depends(get_current_user_id)
):
    try:
        client = Hyperspell(
            api_key=os.environ["HYPERSPELL_API_KEY"],
            user_id=user_id
        )
        response = client.memories.search(
            query=request.query,
            answer=True,
            sources=["gmail", "slack"]  # Replace with your providers
        )
        return SearchResponse(answer=response.answer)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Search failed: {str(e)}")


# GET /api/hyperspell/connect - Redirect to Hyperspell Connect (FLOW B)
from fastapi.responses import RedirectResponse
from fastapi import Request

@router.get("/connect")
async def connect(
    request: Request,
    user_id: str = Depends(get_current_user_id)
):
    try:
        client = Hyperspell(api_key=os.environ["HYPERSPELL_API_KEY"])
        response = client.auth.user_token(user_id=user_id)

        base_url = str(request.base_url).rstrip("/")
        redirect_uri = f"{base_url}/api/hyperspell/callback"
        connect_url = f"https://connect.hyperspell.com?token={response.token}&redirect_uri={redirect_uri}"

        return RedirectResponse(url=connect_url)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to initiate connection: {str(e)}")


# GET /api/hyperspell/callback - Handle redirect after connection
@router.get("/callback")
async def callback():
    # User has connected their accounts
    # Redirect to your app's success page
    return RedirectResponse(url="/settings?connected=true")
```

## Direct Memory Operations

For FLOW C (backend with direct memory management), add these endpoints:

```python
# routers/hyperspell.py (add to existing router)

# POST /api/hyperspell/memories/add - Add a memory
@router.post("/memories/add")
async def add_memory(
    request: AddMemoryRequest,
    user_id: str = Depends(get_current_user_id)
):
    try:
        client = Hyperspell(
            api_key=os.environ["HYPERSPELL_API_KEY"],
            user_id=user_id
        )
        response = client.memories.add(
            content=request.content,
            title=request.title,
            source="custom",
            metadata=request.metadata or {}
        )
        return response
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to add memory: {str(e)}")


# POST /api/hyperspell/memories/upload - Upload a file
@router.post("/memories/upload")
async def upload_file(
    file: UploadFile = File(...),
    user_id: str = Depends(get_current_user_id)
):
    try:
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
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to upload file: {str(e)}")


# GET /api/hyperspell/memories - List memories
@router.get("/memories", response_model=ListMemoriesResponse)
async def list_memories(
    limit: int = 20,
    offset: int = 0,
    source: Optional[str] = None,
    user_id: str = Depends(get_current_user_id)
):
    try:
        client = Hyperspell(
            api_key=os.environ["HYPERSPELL_API_KEY"],
            user_id=user_id
        )
        response = client.memories.list(
            limit=limit,
            offset=offset,
            source=source
        )
        return response
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Failed to list memories: {str(e)}")
```

## Register the Router

Add the router to your FastAPI app:

```python
# main.py
from fastapi import FastAPI
from routers import hyperspell

app = FastAPI()

# Include Hyperspell router
app.include_router(hyperspell.router)

# ... rest of your app
```

## Environment Variables

Make sure to set in your environment or `.env`:

```
HYPERSPELL_API_KEY=hs-0-your-api-key
```

If using python-dotenv:

```python
from dotenv import load_dotenv
load_dotenv()
```

## Auth Dependency Examples

### FastAPI Users

```python
from fastapi_users import FastAPIUsers
from models import User

fastapi_users = FastAPIUsers[User, int](
    get_user_manager,
    [auth_backend],
)

current_active_user = fastapi_users.current_user(active=True)

async def get_current_user_id(user: User = Depends(current_active_user)) -> str:
    return str(user.id)
```

### JWT Authentication

```python
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

async def get_current_user_id(token: str = Depends(oauth2_scheme)) -> str:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401, detail="Invalid token")
        return user_id
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")
```

### API Key Authentication

```python
from fastapi.security import APIKeyHeader

api_key_header = APIKeyHeader(name="X-API-Key")

async def get_current_user_id(api_key: str = Depends(api_key_header)) -> str:
    # Validate API key and get associated user
    user = validate_api_key(api_key)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid API key")
    return user.id
```

## CORS Configuration

If your frontend is on a different domain:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # Your frontend URL
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## Frontend Integration

If you have a separate frontend:

```typescript
// Frontend code
const API_BASE = 'http://localhost:8000';  // Your FastAPI server

async function getHyperspellToken(): Promise<string> {
  const response = await fetch(`${API_BASE}/api/hyperspell/token`, {
    method: 'POST',
    credentials: 'include',
    headers: {
      'Authorization': `Bearer ${getAuthToken()}`,
    },
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
