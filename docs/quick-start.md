If you are using SQLModel, go to [Using SQLModel](sqlmodel.md) instead.

## Minimal Example

Assuming you have your SQLAlchemy model, Pydantic schemas and database connection, just skip to [Using FastCRUD](#using-fastcrud)

### Basic Setup

Define your SQLAlchemy model

```python title="setup.py" hl_lines="11-19"
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import DeclarativeBase, sessionmaker
from sqlalchemy.ext.asyncio import AsyncSession
from pydantic import BaseModel


class Base(DeclarativeBase):
    pass


--8<--
fastcrud/examples/item/model.py:model
fastcrud/examples/item/schemas.py:itemschema
--8<--
DATABASE_URL = "sqlite+aiosqlite:///./test.db"
engine = create_async_engine(DATABASE_URL, echo=True)
async_session = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
```

Then your Pydantic schemas

```python title="setup.py" hl_lines="22-27"
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import DeclarativeBase, sessionmaker
from sqlalchemy.ext.asyncio import AsyncSession
from pydantic import BaseModel


class Base(DeclarativeBase):
    pass


--8<--
fastcrud/examples/item/model.py:model
fastcrud/examples/item/schemas.py:itemschema
--8<--
DATABASE_URL = "sqlite+aiosqlite:///./test.db"
engine = create_async_engine(DATABASE_URL, echo=True)
async_session = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
```

And, finally, your database connection

```python title="setup.py" hl_lines="30-32"
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import DeclarativeBase, sessionmaker
from sqlalchemy.ext.asyncio import AsyncSession
from pydantic import BaseModel


class Base(DeclarativeBase):
    pass


--8<--
fastcrud/examples/item/model.py:model
fastcrud/examples/item/schemas.py:itemschema
--8<--
DATABASE_URL = "sqlite+aiosqlite:///./test.db"
engine = create_async_engine(DATABASE_URL, echo=True)
async_session = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)
```

### Using FastCRUD

Use `crud_router` and include it in your `FastAPI` application

```python title="main.py" hl_lines="17-24 26"
from fastcrud import crud_router

# Database session dependency
async def get_session() -> AsyncGenerator[AsyncSession, None]:
    async with async_session() as session:
        yield session

# Create tables before the app start
async def lifespan(app: FastAPI):
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield

# FastAPI app
app = FastAPI(lifespan=lifespan)

item_router = crud_router(
    session=session,
    model=Item,
    create_schema=ItemSchema,
    update_schema=ItemSchema,
    path="/items",
    tags=["Items"]
)

app.include_router(item_router)
```

And it's all done, just go to `/docs` and the crud endpoints are created.
