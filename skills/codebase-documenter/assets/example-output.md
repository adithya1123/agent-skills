# Example Output: FastAPI + PostgreSQL Project

This example shows what a complete AGENTS/ directory looks like for a
typical FastAPI + SQLAlchemy + PostgreSQL project.

---

## AGENTS/00_map.md (example)

```markdown
# Codebase Map
_Last updated: 2026-04-25_

## Quick Reference
- **Stack**: Python 3.11, FastAPI 0.110, SQLAlchemy 2.0, PostgreSQL 15, Redis
- **Entry points**: `app/main.py` → `app/api/router.py` (all routes registered here)
- **Test command**: `pytest -x --cov=app tests/`
- **Deploy command**: `make deploy` (runs migrations then restarts gunicorn)

## Module Index

| Module | Path | Purpose | Key contracts |
|--------|------|---------|---------------|
| auth | `app/auth/` | JWT issue/verify, session management | `→ contracts/auth.md` |
| users | `app/users/` | CRUD, role assignment | `→ contracts/users.md` |
| billing | `app/billing/` | Stripe integration, invoice generation | `→ contracts/billing.md` |
| jobs | `app/jobs/` | Celery background tasks | `→ contracts/jobs.md` |
| db | `app/db/` | Session factory, base models | `→ contracts/db.md` |

## Data Model Summary
Core entities: `User` (has many `Subscription`s, one active at a time), 
`Organization` (Users belong to one), `Invoice` (belongs to Subscription).
Primary key convention: UUID4 everywhere, never int. Foreign keys use `ON DELETE RESTRICT`
— deleting a User with active Subscriptions raises `IntegrityError`, not a cascade.

→ For full contracts: `contracts/`
→ For hazards: `01_hazards.md`
→ For task playbooks: `playbooks/`
```

---

## AGENTS/contracts/auth.md (example)

```markdown
# Contracts: Auth
_Last updated: 2026-04-25_
_Covers: `app/auth/service.py`, `app/auth/dependencies.py`_

## get_current_user (FastAPI dependency)

**Summary**: Resolves the JWT in `Authorization: Bearer` header to a User ORM object.
**File**: `app/auth/dependencies.py:18`

**Returns**:
- `User` ORM object (SQLAlchemy model, not Pydantic schema)
- Raises `HTTPException(401)` if token missing or expired
- Raises `HTTPException(403)` if user is deactivated (status != "active")
- Does NOT raise if user has unpaid invoices — billing checks are separate

**Produces**: `User` ORM object — the authenticated user injected into every protected route handler

**Consumed by**: Every route handler that declares `current_user: User = Depends(get_current_user)`

**Invariants**:
- Injects db session via `Depends(get_db)` — do not pass db separately
- The returned User already has `organization` relationship eagerly loaded

**Side effects**: None. Read-only.

**Idempotency**: SAFE

→ See also: `playbooks/add_endpoint.md`, `01_hazards.md#auth`

---

## AuthService.issue_token(user_id, scopes)

**Summary**: Issues a signed JWT. Call this only after verifying credentials.
**File**: `app/auth/service.py:44`

**Non-obvious inputs**:
- `scopes`: list of strings, must be subset of `VALID_SCOPES` in `app/config.py`
  Passing invalid scope raises `ValueError`, not an HTTP exception
- Does NOT accept a User object — only `user_id` UUID

**Returns**: `{"access_token": str, "expires_in": int}` — no refresh token here,
  refresh is handled by `AuthService.refresh_token()`

**Produces**: `access_token` JWT string — the bearer token returned to the client on login

**Consumed by**:
- `api/v2/auth/login` route — calls this after credential verification
- `api/v2/auth/refresh` route — calls `AuthService.refresh_token()` which calls this internally

**Side effects**:
- Writes token hash to Redis with TTL=`settings.JWT_TTL_SECONDS`
- This means token revocation works — blacklisting via Redis

**Env dependencies**:
- Reads `JWT_SECRET` at import time (not per-call) — app crashes on startup if missing
```

---

## AGENTS/01_hazards.md (example)

```markdown
# Hazard Map
_Last updated: 2026-04-25_

## 🔴 NEVER

### Never use `db.query(User)` — use `UserRepository`
**What**: Raw SQLAlchemy queries on User bypass soft-delete filter.
**Instead**: `UserRepository(db).get_by_id(id)` — automatically filters `deleted_at IS NULL`.
**Why it matters**: Deleted users would appear in results, leaking PII.

### Never call `stripe.Charge.create()` directly
**What**: Direct Stripe calls bypass idempotency key logic in `BillingService`.
**Instead**: `BillingService(db).charge(subscription_id, amount_cents)`.
**Why it matters**: Duplicate charges on retry without idempotency keys.

---

## 🟡 CAUTION

### Celery tasks must use `bind=True` and handle `autoretry_for`
**Where**: All tasks in `app/jobs/`
**Detail**: Task failures without retry config are silently swallowed in prod.
**Pattern**:
```python
@app.task(bind=True, autoretry_for=(Exception,), max_retries=3, default_retry_delay=60)
def my_task(self, ...):
```

### SQLAlchemy 2.0: `session.execute()` returns `CursorResult`, not list
**Where**: Any raw SQL
**Detail**: Must call `.scalars().all()` or `.mappings().all()` — `.fetchall()` 
  returns Row objects, not dicts.

### Redis connection pool is shared — don't close it
**Where**: `app/cache/client.py`
**Detail**: `redis_client` is a module-level singleton. Calling `.close()` anywhere
  kills it for the entire process.

---

## ⚪ CONVENTION

### All new API routes go in `app/api/v2/`
v1 (`app/api/v1/`) is deprecated. New endpoints only in v2.

### Pydantic response schemas live in `app/schemas/`, NOT in the route file
The route file imports the schema — it never defines one inline.

### Use `structlog.get_logger()` not `logging.getLogger()`
Middleware expects structlog context vars — standard logging misses request_id injection.
```

---

## AGENTS/playbooks/add_endpoint.md (example)

```markdown
# Playbook: Add a New API Endpoint
_Last updated: 2026-04-25_

## Steps

1. **Create the route function** in `app/api/v2/{resource}.py`
   ```python
   @router.post("/{resource}", response_model=schemas.{Resource}Response, status_code=201)
   async def create_{resource}(
       payload: schemas.{Resource}Create,
       db: AsyncSession = Depends(get_db),
       current_user: User = Depends(get_current_user),
   ):
       return await {Resource}Service(db).create(payload, owner=current_user)
   ```

2. **Create request/response schemas** in `app/schemas/{resource}.py`
   ```python
   class {Resource}Create(BaseModel):
       name: str
       
   class {Resource}Response(BaseModel):
       id: UUID
       name: str
       created_at: datetime
       
       model_config = ConfigDict(from_attributes=True)
   ```

3. **Register the router** in `app/api/router.py`
   ```python
   from app.api.v2 import {resource}
   api_router.include_router({resource}.router, prefix="/{resources}", tags=["{Resource}"])
   ```

4. **Write a test** in `tests/api/v2/test_{resource}.py`
   ```python
   async def test_create_{resource}(client, auth_headers):
       resp = await client.post("/{resources}", json={...}, headers=auth_headers)
       assert resp.status_code == 201
   ```

## Validation
```bash
pytest tests/api/v2/test_{resource}.py -v
```

## Common failures

**`422 Unprocessable Entity` in tests**: Pydantic schema mismatch — check field names.
**`ImportError` on startup**: Router not registered in `app/api/router.py`.
**`AttributeError: 'NoneType'`**: `get_current_user` dependency not injected.

→ See also: `contracts/auth.md#get_current_user`, `01_hazards.md#structlog`
```
