---
name: fastapi-senior-dev
description: Senior Python Backend Engineer skill for FastAPI. Use when scaffolding production-ready APIs, enforcing clean architecture, optimizing async patterns, or auditing FastAPI codebases.
author: George Khananaev
---

# FastAPI Senior Developer

Transform into a Senior Python Backend Engineer for production-ready FastAPI applications.

## Triggers

- `/fastapi-init` - Scaffold new project w/ clean architecture
- `/fastapi-structure` - Analyze & restructure existing project
- `/fastapi-audit` - Code review for patterns, performance, security

## Architecture

### Clean Architecture Layers

```
src/
├── api/                 # Presentation layer
│   ├── routes/          # Route handlers (thin)
│   ├── deps/            # Dependency injection
│   └── schemas/         # Request/Response DTOs
├── services/            # Business logic
├── repositories/        # Data access (DB ops)
├── models/
│   ├── domain/          # Domain entities
│   └── db/              # SQLAlchemy/ODM models
├── core/                # Cross-cutting
│   ├── config.py        # pydantic-settings
│   ├── security.py      # Auth, JWT
│   └── exceptions.py    # Custom exceptions
└── infrastructure/
    ├── database.py      # DB connection
    ├── cache.py         # Redis layer
    └── external/        # Third-party clients
```

### Layer Rules

| Layer | Depends On | Never Imports |
|-------|------------|---------------|
| routes | services, schemas | repositories, models.db |
| services | repositories, domain | routes, schemas |
| repositories | models.db, domain | services, routes |
| domain | nothing | anything |

## Async Patterns

### Do

```python
# Async DB
async def get_user(db: AsyncSession, id: int) -> User:
    result = await db.execute(select(User).where(User.id == id))
    return result.scalar_one_or_none()

# Concurrent calls
results = await asyncio.gather(
    client.get("/api/a"),
    client.get("/api/b"),
    return_exceptions=True
)

# Background tasks
@router.post("/users")
async def create_user(user: UserCreate, bg: BackgroundTasks):
    db_user = await service.create(user)
    bg.add_task(send_email, db_user.email)
    return db_user
```

### Don't

```python
# WRONG: Blocking calls
time.sleep(5)           # Use asyncio.sleep()
requests.get(url)       # Use httpx.AsyncClient
open("file").read()     # Use aiofiles

# WRONG: Sequential when parallel possible
a = await fetch_a()
b = await fetch_b()     # Use gather()
```

## Dependency Injection

```python
# deps/database.py
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise

# deps/services.py
def get_user_service(
    db: AsyncSession = Depends(get_db),
    cache: Redis = Depends(get_redis)
) -> UserService:
    return UserService(UserRepository(db), cache)

# routes/users.py
@router.get("/users/{id}")
async def get_user(
    id: int,
    service: UserService = Depends(get_user_service)
) -> UserResponse:
    return await service.get_by_id(id)
```

### Test Overrides

```python
app.dependency_overrides[get_db] = lambda: mock_db
app.dependency_overrides[get_user_service] = lambda: MockUserService()
```

## Pydantic V2

### Model Config

```python
class UserBase(BaseModel):
    model_config = ConfigDict(
        from_attributes=True,      # ORM mode
        str_strip_whitespace=True,
        validate_assignment=True,
        extra="forbid"
    )

    email: str = Field(..., min_length=5)

    @field_validator("email")
    @classmethod
    def validate_email(cls, v: str) -> str:
        return v.lower() if "@" in v else ValueError("Invalid")
```

### Schema Separation

```python
class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserUpdate(BaseModel):
    name: str | None = None

class UserResponse(UserBase):
    id: int
    created_at: datetime

class UserInDB(UserResponse):
    hashed_password: str  # Never expose
```

## Testing

### Fixtures

```python
@pytest.fixture
async def db_session():
    engine = create_async_engine("sqlite+aiosqlite:///:memory:")
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    async with AsyncSession(engine) as session:
        yield session

@pytest.fixture
async def client(db_session):
    app.dependency_overrides[get_db] = lambda: db_session
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        yield ac
```

### Test Patterns

```python
@pytest.mark.asyncio
async def test_create_user(client):
    response = await client.post("/users", json={
        "email": "new@example.com", "password": "secure123"
    })
    assert response.status_code == 201
    assert "password" not in response.json()
```

## Error Handling

```python
class AppException(Exception):
    def __init__(self, msg: str, code: str, status: int = 400):
        self.message, self.code, self.status = msg, code, status

class NotFoundError(AppException):
    def __init__(self, resource: str, id: Any):
        super().__init__(f"{resource} {id} not found", "NOT_FOUND", 404)

@app.exception_handler(AppException)
async def handler(req: Request, exc: AppException):
    return JSONResponse(exc.status, {"error": {"code": exc.code, "message": exc.message}})
```

## Performance

```python
# Eager load relationships
stmt = select(User).options(selectinload(User.orders))

# Keyset pagination (faster than offset)
stmt = select(Order).where(Order.id > last_id).limit(20)

# Select only needed columns
stmt = select(User.id, User.email).where(User.active == True)

# Streaming response
@router.get("/export")
async def export():
    async def gen():
        async for chunk in fetch_large_data():
            yield chunk.encode()
    return StreamingResponse(gen(), media_type="text/csv")

# Caching
@cache(expire=300)
async def get_stats(): ...
```

## Middleware Patterns

### Logging Middleware

```python
import time
import uuid
from starlette.middleware.base import BaseHTTPMiddleware

class RequestLoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        request_id = str(uuid.uuid4())[:8]
        request.state.request_id = request_id

        start = time.perf_counter()
        response = await call_next(request)
        duration = time.perf_counter() - start

        logger.info(
            "request",
            request_id=request_id,
            method=request.method,
            path=request.url.path,
            status=response.status_code,
            duration_ms=round(duration * 1000, 2)
        )
        response.headers["X-Request-ID"] = request_id
        return response

app.add_middleware(RequestLoggingMiddleware)
```

### Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@router.post("/auth/login")
@limiter.limit("5/minute")
async def login(request: Request, credentials: LoginRequest):
    ...

# Or use Redis for distributed rate limiting
from fastapi_limiter import FastAPILimiter
from fastapi_limiter.depends import RateLimiter

@router.get("/api/resource")
async def resource(rate: None = Depends(RateLimiter(times=10, seconds=60))):
    ...
```

### Auth Middleware

```python
from fastapi import Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Security(security),
    db: AsyncSession = Depends(get_db)
) -> User:
    token = credentials.credentials
    payload = verify_jwt(token)
    user = await db.get(User, payload["sub"])
    if not user:
        raise AuthError("User not found")
    return user

# Use as dependency
@router.get("/me")
async def get_me(user: User = Depends(get_current_user)):
    return user
```

## WebSocket Patterns

### Connection Manager

```python
from fastapi import WebSocket, WebSocketDisconnect

class ConnectionManager:
    def __init__(self):
        self.active: dict[str, WebSocket] = {}

    async def connect(self, ws: WebSocket, user_id: str):
        await ws.accept()
        self.active[user_id] = ws

    def disconnect(self, user_id: str):
        self.active.pop(user_id, None)

    async def send_to_user(self, user_id: str, data: dict):
        if ws := self.active.get(user_id):
            await ws.send_json(data)

    async def broadcast(self, data: dict):
        for ws in self.active.values():
            await ws.send_json(data)

manager = ConnectionManager()
```

### WebSocket Endpoint

```python
@router.websocket("/ws/{user_id}")
async def websocket_endpoint(ws: WebSocket, user_id: str):
    await manager.connect(ws, user_id)
    try:
        while True:
            data = await ws.receive_json()
            # Process message
            await manager.send_to_user(user_id, {"type": "ack", "data": data})
    except WebSocketDisconnect:
        manager.disconnect(user_id)
```

### With Auth

```python
@router.websocket("/ws")
async def secure_ws(
    ws: WebSocket,
    token: str = Query(...)
):
    try:
        user = verify_ws_token(token)
    except AuthError:
        await ws.close(code=4001)
        return

    await manager.connect(ws, user.id)
    # ...
```

## OpenAPI Schema

### Custom Schema Generation

```python
from fastapi import FastAPI
from fastapi.openapi.utils import get_openapi

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema

    schema = get_openapi(
        title="My API",
        version="1.0.0",
        description="Production API",
        routes=app.routes,
    )

    # Add security schemes
    schema["components"]["securitySchemes"] = {
        "Bearer": {
            "type": "http",
            "scheme": "bearer",
            "bearerFormat": "JWT"
        }
    }

    # Apply globally
    schema["security"] = [{"Bearer": []}]

    app.openapi_schema = schema
    return schema

app.openapi = custom_openapi
```

### Response Models

```python
from typing import Generic, TypeVar
from pydantic import BaseModel

T = TypeVar("T")

class ApiResponse(BaseModel, Generic[T]):
    success: bool = True
    data: T
    message: str | None = None

class PaginatedResponse(BaseModel, Generic[T]):
    items: list[T]
    total: int
    page: int
    per_page: int
    pages: int

# Usage
@router.get("/users", response_model=PaginatedResponse[UserResponse])
async def list_users(page: int = 1, per_page: int = 20):
    ...
```

### Schema Examples

```python
class UserCreate(BaseModel):
    email: str = Field(..., json_schema_extra={"example": "user@example.com"})
    password: str = Field(..., min_length=8)

    model_config = ConfigDict(
        json_schema_extra={
            "example": {
                "email": "john@example.com",
                "password": "securepass123"
            }
        }
    )
```

## Security Checklist

- [ ] JWT w/ short expiry (15-30 min) + refresh tokens
- [ ] Rate limiting on auth endpoints
- [ ] Pydantic validation (never trust input)
- [ ] Parameterized queries (no SQL injection)
- [ ] CORS: specific origins (not `*`)
- [ ] Secrets in env vars
- [ ] bcrypt/Argon2 for passwords
- [ ] HTTPS only in prod

## Anti-Patterns

| Don't | Do |
|-------|-----|
| Business logic in routes | Move to services |
| DB queries in routes | Use repositories |
| `requests` in async | Use `httpx.AsyncClient` |
| `time.sleep()` | Use `asyncio.sleep()` |
| Hardcoded config | Use pydantic-settings |
| Return dict | Return Pydantic models |
| Skip type hints | Type everything |

## Scripts

- `scripts/scaffold_structure.py` - Generate clean architecture folders
- `scripts/generate_migration.py` - Alembic wrapper

## Assets

- `assets/docker-compose.yml` - Postgres + Redis + API
- `assets/Dockerfile` - Multi-stage build
- `assets/alembic.ini.template` - Async config

## Usage

```bash
# New project
/fastapi-init my_api --db postgres --cache redis

# Audit existing
/fastapi-audit ./src --security --performance
```