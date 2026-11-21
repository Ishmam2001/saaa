# Technical Design Document (TDD) — Campus Lost & Found

| Field | Value |
|---|---|
| Product | Campus Lost & Found Management System |
| Releases | MVP — Mid-term; Beta — Final |
| Status | Complete (Beta Released) |
| Related docs | [PRD](01-prd.md), [SRS](02-srs.md) |

---

## 1. Overview

- **MVP:** A local React + TypeScript frontend uses a FastAPI REST API to manage lost/found reports stored in SQLite.
- **Beta:** Added JWT authentication (access & rotating refresh tokens), token revocation blacklist, account lockout (5 failed attempts), RBAC roles (`user`, `moderator`, `admin`), campus team desks, automated test suite, and cloud deployment on Vercel.

---

## 2. Technology

| Layer | Technology | Release |
|---|---|---|
| Frontend | React, TypeScript, Vite | MVP |
| Backend | Python, FastAPI, SQLModel / SQLAlchemy | MVP & Beta |
| API | JSON REST endpoints | MVP & Beta |
| Data storage | SQLite (local dev) / PostgreSQL (production) | MVP & Beta |
| Authentication | JWT (PyJWT), Argon2 / PBKDF2 password hashing | Beta |
| Testing | Pytest (`test_auth_and_rbac.py`) | Beta |
| Deployment | Vercel (frontend + serverless Python backend) | Beta |

---

## 3. Architecture

```text
Browser (React + TypeScript Frontend)
  └── /api requests (HTTPS / JSON)
        └── FastAPI Backend (Serverless on Vercel)
              ├── JWT & RBAC Middleware (security.py, rbac.py)
              └── Database (SQLite / PostgreSQL)
```

For local development, Vite proxies `/api` requests to `localhost:8000`. In production, Vercel routes `/api/*` directly to the FastAPI backend.

---

## 4. Project Structure

```text
backend/
  main.py             # FastAPI entrypoint and router registration
  models.py           # SQLModel database tables and Pydantic schemas
  database.py         # Database engine and session management
  security.py         # Argon2/PBKDF2 hashing, JWT creation/verification, lockout check
  rbac.py             # Role-based access control and dependency checkers
  crud.py             # Database queries for reports, users, tokens, and groups
  service.py          # Business logic, seed data, and ownership checks
  routers/
    items.py          # Report CRUD endpoints with ownership enforcement
    auth.py           # Register, login, refresh, and logout endpoints
    profile.py        # Profile info and password change endpoints
    admin.py          # Admin statistics, user list, and role management
  tests/
    test_auth_and_rbac.py  # 7 automated security, lockout, and RBAC tests

frontend/
  src/
    App.tsx           # Main application state, auth state, and view routing
    api.ts            # REST API client with auto-refresh on 401
    types.ts          # TypeScript interfaces (Item, User, Group)
    components/       # Reusable UI (Header, FilterBar, ItemCard, ItemList, Modals)
    pages/            # View pages (LoginPage, SignupPage, ProfilePage, AdminPage)

docs/
  01-prd.md
  02-srs.md
  03-tdd.md
  FINAL_REPORT.md
```

---

## 5. Data Model

### Report / Item (`Item`)

| Field | Description | Release |
|---|---|---|
| `id` | Unique report identifier | MVP |
| `type` | `lost` or `found` | MVP |
| `title` | Short item title | MVP |
| `description` | Item details | MVP |
| `category` | Item category (IDs, Electronics, Books, etc.) | MVP |
| `location` | Where it was lost or found | MVP |
| `status` | `active` or `resolved` | MVP |
| `created_at` | Creation timestamp | MVP |
| `owner_id` | Foreign key to User account (creator) | Beta |
| `group_id` | Foreign key to Team Desk (optional) | Beta |

### User (`User`) — Beta

| Field | Description |
|---|---|
| `id` | Unique account identifier |
| `name` | Display name |
| `email` | Unique user email |
| `password_hash` | Secure Argon2/PBKDF2 password hash |
| `role` | `user`, `moderator`, or `admin` |
| `is_active` | Account active flag |
| `failed_attempts` | Consecutive failed login counter |
| `locked_until` | Lockout expiration timestamp (15 min lock) |

### Team Desk (`TeamGroup`) — Beta

| Field | Description |
|---|---|
| `id` | Unique desk identifier |
| `name` | Desk name (`Security Desk`, `Library Front Desk`, `IT Helpdesk`) |
| `description` | Desk description |

### Token Blacklist (`RevokedToken`) — Beta

| Field | Description |
|---|---|
| `jti` | Unique JWT identifier |
| `revoked_at` | Revocation timestamp |
| `expires_at` | Token expiration timestamp |

---

## 6. REST API

All endpoints use the `/api` prefix and JSON.

### Report Endpoints

| Method | Path | Access | Purpose |
|---|---|---|---|
| `GET` | `/api/items` | Public | List reports with search & category filters |
| `GET` | `/api/items/{id}` | Public | View one report |
| `POST` | `/api/items` | Authenticated | Create a report (assigns `owner_id`) |
| `PUT` | `/api/items/{id}` | Owner / Staff / Admin | Update report |
| `DELETE` | `/api/items/{id}` | Owner / Staff / Admin | Delete report |
| `GET` | `/api/health` | Public | API health check |

### Authentication & User Endpoints — Beta

| Method | Path | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/auth/register` | Public | Register new member (defaults to `user` role) |
| `POST` | `/api/auth/login` | Public | Authenticate user (checks lockout, returns tokens) |
| `POST` | `/api/auth/refresh` | Public | Rotate refresh token & get new access token |
| `POST` | `/api/auth/logout` | Authenticated | Blacklist token & end session |
| `GET` | `/api/profile` | Authenticated | View signed-in user profile & own reports |
| `PUT` | `/api/profile/password` | Authenticated | Change user password |
| `GET` | `/api/groups` | Authenticated | List campus team desks |
| `GET` | `/api/admin/users` | Admin Only | List users and manage roles |
| `GET` | `/api/admin/stats` | Admin Only | System summary metrics |

---

## 7. Security — Beta

- **Password Hashing:** Passwords hashed with Argon2 or PBKDF2; plain text is never stored.
- **JWT Architecture:** Short-lived access tokens (15 min) and long-lived refresh tokens (7 days).
- **Token Revocation:** Logging out stores the JWT's `jti` in a revocation blacklist table.
- **Brute-Force Lockout:** 5 consecutive failed login attempts locks the account for 15 minutes.
- **RBAC:** 3 roles (`user`, `moderator`, `admin`) enforced via FastAPI dependencies.
- **Ownership Rules:** Regular users can only modify their own items; admins and desk moderators can modify any item.
- **HTTPS:** Enforced in production via Vercel SSL certificates.

---

## 8. Testing

- **MVP:** Manual and API verification of report creation, filtering, update, and deletion.
- **Beta:** Automated test suite in `backend/tests/test_auth_and_rbac.py` covering:
  1. Failed login lockout after 5 consecutive attempts.
  2. Successful login and refresh token rotation.
  3. Rejection of invalid and revoked tokens.
  4. RBAC: User restricted to own reports.
  5. RBAC: Moderator desk access.
  6. RBAC: Admin full access.
  7. RBAC: Denial of unauthorized actions (`401` / `403`).

---

## 9. Deployment and Configuration

### Configuration Settings

| Setting | Use | Release |
|---|---|---|
| `DATABASE_URL` | SQLite path or PostgreSQL connection string | MVP & Beta |
| `JWT_SECRET` | Secret key to sign and verify JWT tokens | Beta |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Access token lifespan (default: 15) | Beta |
| `REFRESH_TOKEN_EXPIRE_DAYS` | Refresh token lifespan (default: 7) | Beta |

Secrets are loaded from environment variables and never committed to source control.
