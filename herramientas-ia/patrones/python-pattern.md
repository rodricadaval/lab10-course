# Python Architecture Patterns

## FastAPI Project Structure

```
myapp/
├── main.py                  # Entry point
├── requirements.txt
├── app/
│   ├── core/
│   │   ├── config.py        # Settings, env vars
│   │   └── security.py      # Auth, JWT logic
│   ├── api/
│   │   ├── __init__.py
│   │   └── v1/
│   │       ├── routes/
│   │       │   └── users.py # Route handlers
│   │       └── endpoints/
│   │           └── users.py
│   ├── schemas/             # Pydantic models (request/response)
│   │   └── user.py
│   ├── models/              # SQLAlchemy models (DB)
│   │   └── user.py
│   ├── services/            # Business logic
│   │   └── user_service.py
│   ├── repositories/        # Data access
│   │   └── user_repository.py
│   └── dependencies.py      # Dependency injection
└── tests/
```

---

## Service Layer Pattern

```python
# app/services/user_service.py
from typing import Optional
from app.schemas.user import UserCreate, UserResponse
from app.repositories.user_repository import UserRepository

class UserService:
    def __init__(self, repo: UserRepository):
        self.repo = repo

    async def create_user(self, user_data: UserCreate) -> UserResponse:
        if await self.repo.get_by_email(user_data.email):
            raise ValueError("Email already exists")

        user = await self.repo.save(user_data)
        return UserResponse.from_orm(user)

    async def get_user(self, user_id: int) -> UserResponse:
        user = await self.repo.get_by_id(user_id)
        if not user:
            raise ValueError("User not found")
        return UserResponse.from_orm(user)
```

---

## Repository Pattern with SQLAlchemy (Async)

```python
# app/repositories/user_repository.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.models.user import User
from app.schemas.user import UserCreate

class UserRepository:
    def __init__(self, session: AsyncSession):
        self.session = session

    async def get_by_id(self, user_id: int) -> Optional[User]:
        result = await self.session.execute(
            select(User).where(User.id == user_id)
        )
        return result.scalars().first()

    async def get_by_email(self, email: str) -> Optional[User]:
        result = await self.session.execute(
            select(User).where(User.email == email)
        )
        return result.scalars().first()

    async def save(self, user_data: UserCreate) -> User:
        user = User(**user_data.dict())
        self.session.add(user)
        await self.session.commit()
        await self.session.refresh(user)
        return user

    async def delete(self, user_id: int) -> bool:
        user = await self.get_by_id(user_id)
        if not user:
            return False
        await self.session.delete(user)
        await self.session.commit()
        return True
```

---

## Dependency Injection with FastAPI Depends

```python
# app/dependencies.py
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.services.user_service import UserService
from app.repositories.user_repository import UserRepository
from app.core.database import get_session

async def get_user_repository(
    session: AsyncSession = Depends(get_session)
) -> UserRepository:
    return UserRepository(session)

async def get_user_service(
    repo: UserRepository = Depends(get_user_repository)
) -> UserService:
    return UserService(repo)
```

```python
# app/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends
from app.schemas.user import UserCreate, UserResponse
from app.services.user_service import UserService
from app.dependencies import get_user_service

router = APIRouter()

@router.post("/users", response_model=UserResponse)
async def create_user(
    user_data: UserCreate,
    service: UserService = Depends(get_user_service)
):
    return await service.create_user(user_data)

@router.get("/users/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    service: UserService = Depends(get_user_service)
):
    return await service.get_user(user_id)
```

---

## Async Patterns (asyncio, aiohttp)

```python
# For concurrent operations
import asyncio
from aiohttp import ClientSession

async def fetch_multiple_users(user_ids: list) -> list:
    async with ClientSession() as session:
        tasks = [fetch_user(session, uid) for uid in user_ids]
        return await asyncio.gather(*tasks)

async def fetch_user(session: ClientSession, user_id: int):
    async with session.get(f"https://api.example.com/users/{user_id}") as resp:
        return await resp.json()

# In FastAPI background tasks
from fastapi import BackgroundTasks

@router.post("/users/bulk")
async def bulk_create(
    users: list,
    background_tasks: BackgroundTasks,
    service: UserService = Depends(get_user_service)
):
    result = await service.create_users(users)
    background_tasks.add_task(send_emails, [u.email for u in result])
    return result
```

---

## Agent Patterns: LangChain + FastAPI

```python
# app/services/agent_service.py
from langchain.agents import initialize_agent, Tool
from langchain.llms import ChatOpenAI
from langchain_community.tools.sql_database.tool import QuerySQLDatabaseTool
from app.core.config import settings

class AgentService:
    def __init__(self, repo: UserRepository):
        self.repo = repo
        self.llm = ChatOpenAI(api_key=settings.OPENAI_API_KEY)

    async def answer_question(self, question: str) -> str:
        tools = [
            Tool(
                name="GetUser",
                func=self.repo.get_by_id,
                description="Get user by ID"
            ),
        ]

        agent = initialize_agent(
            tools,
            self.llm,
            agent="zero-shot-react-description",
            verbose=True
        )

        return agent.run(question)
```

---

## Error Handling with Custom Exceptions

```python
# app/core/exceptions.py
class APIException(Exception):
    def __init__(self, status_code: int, detail: str):
        self.status_code = status_code
        self.detail = detail

class UserNotFoundError(APIException):
    def __init__(self):
        super().__init__(404, "User not found")

class ValidationError(APIException):
    def __init__(self, field: str, message: str):
        super().__init__(400, f"{field}: {message}")

# app/main.py
from fastapi import FastAPI
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(APIException)
async def api_exception_handler(request, exc):
    return JSONResponse(
        status_code=exc.status_code,
        content={"detail": exc.detail}
    )
```

---

## Key Takeaways

- **Layered structure** — routes → services → repositories → models
- **Async first** — use `async/await` for I/O-bound operations
- **Dependency injection** — FastAPI's `Depends` keeps code testable
- **Repository pattern** — isolate data access logic
- **Error handling** — custom exceptions for domain errors
- **Agents** — LangChain provides tools and prompting patterns