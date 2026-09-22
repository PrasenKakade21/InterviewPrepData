# SQLAlchemy and Alembic — Revision Notes (Intermediate)
> One-line mental model: Models map tables → sessions stage objects → Alembic versions schema.
## Core recall
- Engine = connection (once); Session = workspace per request (`sessionmaker`, `yield` in FastAPI).
- Model: `class User(Base)` + `__tablename__`, `Mapped[int] = mapped_column(...)`; `create_all()` dev only.
- Columns: `primary_key/unique/nullable/default`; mirrors SQL constraints.
- Relationships: `ForeignKey("users.id")` + `relationship(back_populates=...)`; `user.orders` no SQL.
- Session flow: `add()` stage → `commit()` flush → `rollback()` on error; `refresh()` loads new id.
- Query 2.0: `select(User).where(User.age>=18).order_by().limit()`; `session.scalars(stmt).all()`, `session.get(User,1)`.
- CRUD: `add/commit`, `get`, assign + commit, `delete` + commit.
- 1-1: FK with `unique=True`; 1-N: parent list + child FK; N-N: link `Table` + `secondary=` (full model if extra cols).
- N+1: lazy loads per-access (slow); eager `selectinload/joinedload` fixes: `select(User).options(selectinload(User.orders))`.
- Alembic: `init` once → edit model → `revision --autogenerate -m` → review → `upgrade head`; `downgrade` reverts.
## Must-remember code
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
engine = create_engine("sqlite:///app.db")
Session = sessionmaker(bind=engine)
```
```python
from sqlalchemy import select
stmt = select(User).where(User.name == "Kakade").limit(10)
users = session.scalars(stmt).all()
```
```python
u = User(name="Kakade"); session.add(u); session.commit(); session.refresh(u)
u.name = "Kakade Dev"; session.commit()
```
```python
from sqlalchemy.orm import selectinload
stmt = select(User).options(selectinload(User.orders))
users = session.scalars(stmt).all()
```
```bash
alembic revision --autogenerate -m "add email"
alembic upgrade head
```
## Interview traps
- `create_all()` in prod loses history → Alembic with reviewable migrations.
- Lazy loop `u.orders` = N+1 queries → eager load when needed.
- Forgetting `rollback()` after failed commit poisons session → try/except + rollback.
- N-N link with extra cols (grade) as plain `secondary` loses data → full association model.
- Forgetting `refresh()` → new `id` appears missing on object.
## 3 things to remember
1. Map tables to classes, query with `select()`.
2. Relate: 1-1 unique FK, 1-N list, N-N link table.
3. Migrate safely: autogenerate → review → upgrade head.
