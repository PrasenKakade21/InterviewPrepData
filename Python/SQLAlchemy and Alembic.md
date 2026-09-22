# SQLAlchemy and Alembic

Raw SQL is explicit. SQLAlchemy maps **tables → Python classes** so you query with objects.

The core idea:

```text
Table users
    ↓
class User(Base)
    ↓
session queries objects
    ↓
commit saves, Alembic migrates schema
```

Install:

```bash
pip install sqlalchemy alembic
```

---

# 1. SQLAlchemy Basics

Two parts: `Engine` (connection) + `Session` (workspace).

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine("sqlite:///app.db")
Session = sessionmaker(bind=engine)
session = Session()
```

Think:

```text
Engine → phone line to DB
Session → conversation (query, add, commit)
```

For interviews: engine is created once, session per request.

---

# 2. Models

A model is a class = table.

```python
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import Integer, String
from sqlalchemy.orm import Mapped, mapped_column


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
```

Creates:

```sql
CREATE TABLE users (id INTEGER PRIMARY KEY, name VARCHAR(50));
```

Create tables (dev only):

```python
Base.metadata.create_all(engine)
```

Production uses Alembic, not `create_all`.

---

# 3. Columns

Column types + rules.

```python
from sqlalchemy import String, Integer, Boolean, DateTime
from sqlalchemy.orm import mapped_column


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(100), unique=True, nullable=False)
    age: Mapped[int] = mapped_column(default=18)
    active: Mapped[bool] = mapped_column(default=True)
```

| Param | Meaning |
| ----- | ------- |
| `primary_key=True` | PK |
| `unique=True` | No duplicates |
| `nullable=False` | Required |
| `default=` | Python-side default |

Matches SQL constraints from previous file.

---

# 4. Relationships

Link models without writing JOINs manually.

```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]

    orders: Mapped[list["Order"]] = relationship(back_populates="user")


class Order(Base):
    __tablename__ = "orders"

    id: Mapped[int] = mapped_column(primary_key=True)
    amount: Mapped[int]
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))

    user: Mapped["User"] = relationship(back_populates="orders")
```

Think:

```text
Order.user_id → FK to users.id
User.orders → list of Order
Order.user → parent User
```

Now `kakade.orders` works, no SQL needed.

---

# 5. Sessions

Session tracks changes, then flushes on commit.

```python
session = Session()

user = User(name="Kakade")
session.add(user)
session.commit()
```

Flow:

```text
session.add(obj)
    ↓
staged in memory
    ↓
session.commit()
    ↓
INSERT executed
```

On error:

```python
try:
    session.commit()
except Exception:
    session.rollback()
```

FastAPI pattern: one session per request via `Depends(get_db)`, closed with `yield`.

---

# 6. Queries

Modern 2.0 style uses `select()`.

```python
from sqlalchemy import select

stmt = select(User).where(User.name == "Kakade")
users = session.scalars(stmt).all()

print(users)
```

Output:

```text
[<User Kakade>]
```

Filter, order, limit:

```python
stmt = (
    select(User)
    .where(User.age >= 18)
    .order_by(User.name)
    .limit(10)
)
```

Equivalent SQL:

```sql
SELECT * FROM users WHERE age >= 18 ORDER BY name LIMIT 10;
```

---

# 7. CRUD Operations

Create:

```python
user = User(name="Kakade")
session.add(user)
session.commit()
session.refresh(user)

print(user.id)
```

Read:

```python
user = session.get(User, 1)
```

Update:

```python
user.name = "Kakade Dev"
session.commit()
```

Delete:

```python
session.delete(user)
session.commit()
```

`session.refresh()` reloads DB defaults (like new `id`) into object.

---

# 8. One-to-One Relationships

One user → one profile. Enforce with `unique=True` on FK.

```python
class Profile(Base):
    __tablename__ = "profiles"

    id: Mapped[int] = mapped_column(primary_key=True)
    bio: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), unique=True)

    user: Mapped["User"] = relationship(back_populates="profile")
```

```text
User(1) ↔ Profile(user_id=1, unique)
```

One row each side. Example: user ↔ settings.

---

# 9. One-to-Many Relationships

Most common: one user → many orders (section 4 example).

```python
user = session.get(User, 1)

print(user.orders)
```

Output:

```text
[<Order 100>, <Order 200>]
```

Add via relationship:

```python
user.orders.append(Order(amount=300))
session.commit()
```

SQLAlchemy fills `user_id` automatically.

---

# 10. Many-to-Many Relationships

Students ↔ courses. Needs association table.

```python
from sqlalchemy import Table, Column


enrollment = Table(
    "enrollment",
    Base.metadata,
    Column("user_id", ForeignKey("users.id"), primary_key=True),
    Column("course_id", ForeignKey("courses.id"), primary_key=True),
)


class Course(Base):
    __tablename__ = "courses"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    students: Mapped[list["User"]] = relationship(
        secondary=enrollment, back_populates="courses"
    )
```

```text
User ↔ enrollment ↔ Course
```

Use `secondary=` for pure link tables. If link needs extra columns (grade, date), make it a full model.

---

# 11. Eager vs Lazy Loading

Lazy (default): loads relationships on access. Can cause N+1 queries.

```python
users = session.scalars(select(User)).all()
for u in users:
    print(u.orders)  # extra query per user!
```

Eager: load everything in one query.

```python
from sqlalchemy.orm import selectinload

stmt = select(User).options(selectinload(User.orders))
users = session.scalars(stmt).all()
```

Think:

```text
lazy → 1 + N queries (slow in loops)
eager → 1-2 queries (fast)
```

Interview answer: "Use `selectinload`/`joinedload` when you know you'll need relationships."

---

# 12. Transactions

Same as SQL: all or nothing.

```python
try:
    session.add(User(name="Kakade"))
    session.add(Order(amount=100))
    session.commit()
except Exception:
    session.rollback()
    raise
```

Transfer example:

```python
a.balance -= 100
b.balance += 100
session.commit()
```

If second line fails, `rollback()` undoes first. Never leave partial writes.

---

# 13. Alembic

Alembic versions your schema like git versions code. Never edit production tables by hand.

Setup:

```bash
alembic init alembic
```

Flow:

```text
1. Edit models (add column)
       ↓
2. alembic revision --autogenerate -m "add email"
       ↓
3. alembic upgrade head
```

Generated migration:

```python
def upgrade():
    op.add_column("users", sa.Column("email", sa.String(100)))


def downgrade():
    op.drop_column("users", "email")
```

`upgrade` applies, `downgrade` reverts. Always review autogenerate before applying.

---

# 14. Database Migrations

Migration rules for interviews:

```text
create_all() → dev/tests only
Alembic → production, reviewable, reversible
```

Typical pipeline:

```text
change model
    ↓
revision --autogenerate
    ↓
review file
    ↓
upgrade head (dev → staging → prod)
```

Never delete old migrations. They are history.

---

# Real-World Example

```python
from sqlalchemy import select

def get_user_with_orders(session, user_id: int):
    from sqlalchemy.orm import selectinload
    stmt = (
        select(User)
        .where(User.id == user_id)
        .options(selectinload(User.orders))
    )
    return session.scalars(stmt).first()
```

FastAPI route calls service, service queries with eager load, Pydantic `UserOut` filters response.

---

# Quick Revision

| Concept | Remember |
| ------- | -------- |
| **Engine/Session** | Connect once, session per request |
| **Models** | Class = table, `__tablename__` |
| **Columns** | Types + `unique/nullable/default` |
| **Relationships** | `ForeignKey` + `relationship` |
| **Session** | `add/commit/rollback/refresh` |
| **Queries** | `select(Model).where()` |
| **CRUD** | `get/add/delete/commit` |
| **1-1** | FK with `unique=True` |
| **1-N** | Parent list + child FK |
| **N-N** | Association table + `secondary` |
| **Eager/Lazy** | `selectinload` avoids N+1 |
| **Transactions** | All or rollback |
| **Alembic** | `revision` + `upgrade head` |

## The Mental Model

```text
        MODELS (classes)
             │
             ↓
   Columns + FK + relationship
             │
   ┌─────────┼─────────┐
   ↓         ↓         ↓
  1-1       1-N       N-N
   │         │         │
   └─────────┼─────────┘
             ↓
   Session (add/query/commit)
             │
             ↓
   select() + eager load
             │
             ↓
   Alembic versions schema
```

### The 3 things to remember

**1. Map then query:**

```python
class User(Base):
    __tablename__ = "users"

session.scalars(select(User)).all()
```

**2. Relate correctly:**

```text
1-1 unique FK, 1-N parent list, N-N link table
```

**3. Migrate safely:**

```bash
alembic revision --autogenerate -m "msg"
alembic upgrade head
```

> **SQLAlchemy turns rows into objects: define models with relationships, work through sessions, eager-load to avoid N+1, and evolve schema with Alembic.**
