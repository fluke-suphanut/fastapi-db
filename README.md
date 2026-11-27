# fastapi-db

![Banner: a large key under a red lightning bolt, next to an open padlock and a database stack](docs/banner.png)

An async FastAPI REST API with JWT login, users and items stored in PostgreSQL.

Portfolio: https://suphanut.pages.dev

## What it does

All API routes are under `/api/v1`.

- `POST /auth/register` creates a user and hashes the password with bcrypt.
- `POST /auth/login` checks an email and password and returns a JWT access token and a refresh token.
  It allows 5 requests per minute per client IP address.
- `POST /auth/refresh-token` returns a new access token for a refresh token.
- `GET` and `POST /users/`, and `GET /users/{user_id}`, list, create and read users.
- `GET /items/` lists items. `POST /items/{user_id}/items/` creates an item for a user.
- The users and items routes need a bearer token and reject inactive users.
- Alembic migrations create the `users` and `items` tables.
- `/scalar` serves a Scalar API reference, next to the FastAPI docs at `/docs`.

## Why

No file states the purpose. Thai code comments explain each step, such as how
Alembic finds the models, so the project reads as a FastAPI and async SQLAlchemy study project.

## Tech stack

- Python 3.14, managed with uv
- FastAPI and Uvicorn
- SQLAlchemy 2 (async) with asyncpg for PostgreSQL
- Alembic for migrations
- Pydantic, pydantic-settings, python-jose (JWT), bcrypt, SlowAPI and scalar-fastapi

## How to run

1. Start a PostgreSQL server and create an empty database.
2. Create a `.env` file in the repository root with these variables:
   - `DATABASE_URL` (required): an async SQLAlchemy URL that uses the `postgresql+asyncpg://` scheme.
   - `SECRET_KEY` (required): the key that signs the JWTs.
   - `ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `REFRESH_TOKEN_EXPIRE_DAYS` (optional).
3. Install the dependencies:

   ```bash
   uv sync
   ```

   Without uv, use `pip install -r requirements.txt` in a Python 3.14 virtual environment.
4. Create the tables:

   ```bash
   uv run alembic upgrade head
   ```

5. Start the server:

   ```bash
   uv run uvicorn app.main:app --reload
   ```

6. Open `http://localhost:8000/scalar` or `http://localhost:8000/docs`.

## Example

Register a user, log in, then list users with the access token:

```bash
curl -X POST http://localhost:8000/api/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "ada@example.com", "password": "example-pass-123", "first_name": "Ada", "last_name": "Lee"}'

curl -X POST http://localhost:8000/api/v1/auth/login \
  -d "username=ada@example.com&password=example-pass-123"

curl http://localhost:8000/api/v1/users/ \
  -H "Authorization: Bearer <access_token>"
```

The login call returns `access_token`, `refresh_token` and `token_type`.

## Limits

- Any logged-in user can list and read all users and create items for any `user_id`.
- The code does not check the token `type` claim. Protected routes accept a refresh
  token, and the refresh endpoint accepts an access token.
- The server does not store or revoke refresh tokens. `POST /auth/logout` only returns a message.
- The API has no update or delete routes.
- The schema marks `first_name` and `last_name` as optional, but the database requires them.
  A request without them fails with a database error.
- The rate limiter keeps its counts in memory, so they reset on restart and are not shared across processes.
- The engine logs every SQL statement (`echo=True`).
- The repository has no tests and no Dockerfile.

## Licence

This repository has no licence file, so default copyright applies.
