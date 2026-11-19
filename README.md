# Campus Lost & Found Management System

A full-stack multi-user web application for reporting, tracking, and recovering lost and found belongings across campus. Built with **FastAPI**, **SQLModel**, **React**, **TypeScript**, and **Vite**, deployed on **Vercel**.

---

## Live Deployment Links

| Resource | URL |
| :--- | :--- |
| **Vercel Frontend App** | [https://campus-lost-found-management-system.vercel.app](https://campus-lost-found-management-system.vercel.app) |
| **FastAPI Interactive API Docs** | [https://campus-lost-found-management-system.vercel.app/api/docs](https://campus-lost-found-management-system.vercel.app/api/docs) |
| **Health Check Endpoint** | [https://campus-lost-found-management-system.vercel.app/api/health](https://campus-lost-found-management-system.vercel.app/api/health) |
| **Final Report (Markdown)** | [docs/FINAL_REPORT.md](docs/FINAL_REPORT.md) |

---

## Features

### 1. Core Lost & Found CRUD
- Create, list, filter (by type, category, status, and search query), view single details, update status (`active`, `claimed`, `resolved`), and delete lost/found reports through `/api/items`.

### 2. Authentication, JWT & Session Management
- **Password Security:** Passwords are combined with a server-side `PASSWORD_PEPPER` (HMAC-SHA256) and hashed with Argon2 (`pwdlib[argon2]`). Plain-text passwords are never stored or returned.
- **Short-Lived JWT Access Tokens:** Signed HS256 JWT (`<= 15 minutes`) containing user ID (`sub`), email, and role. Verified on every protected endpoint (`401 Unauthorized` on missing, invalid, tampered, or expired tokens).
- **Refresh Cookie & Session Control:** Server-side hashed refresh tokens stored in `campus_session` with `HttpOnly` cookie rotation (`POST /api/auth/refresh`), true session invalidation on logout (`POST /api/auth/logout`), and a Profile session manager allowing users to list active devices and revoke individual, other, or all sessions.
- **Brute-Force Lockout:** 5 consecutive failed login attempts lock the account for 15 minutes (`429 Too Many Requests`).

### 3. Role-Based Access Control (RBAC), Ownership & Shared Recovery Teams
- **Centralized RBAC Module (`backend/rbac.py`):** Defines permission constants (`Permission`), role-to-permission sets (`user`, `admin`), reusable FastAPI dependency factory `require_permission(...)`, ownership enforcement, and group permission checks.
- **Ownership Rules:** Normal users can only edit or delete their own reports (`403 Forbidden` otherwise).
- **Shared Recovery Teams (`DeskGroup` / `/api/groups`):** Users can create shared campus recovery teams (e.g., *Main Library Lost & Found Team*), add members by email, and assign granular permissions per member (`can_create`, `can_edit`, `can_assign`). Only the group owner can delete group reports, rename/delete the group, or manage members.
- **Admin Safety Rules:** Admins cannot change their own role or ban another admin (`400 Bad Request`). Banned users are rejected on their very next request (`403 Forbidden`) and logged out immediately.

### 4. Admin Dashboard, Audit Log, Security View & CSV Exports
- **Overview Statistics:** Total users, admins, banned users, total reports (lost/found/claimed), recovery groups, and new users/reports in the last 7 days.
- **User Management:** Search, role/status filters, pagination, user detail inspector modal (showing a user's reports, groups, active sessions, and recent activity), role promotion/demotion, ban/unban with reason, session revocation, and bulk ban/unban.
- **Content Moderation:** Filter and moderate all reports and recovery groups, including single and bulk delete.
- **Activity (Audit) Log & Security Monitor:** Records login, failed login, logout, signup, password change, role change, ban/unban, delete, export, token rejection, and RBAC denial events with user email, IP address, browser User-Agent, and timestamp. Summarizes failed logins by IP address and by account.
- **CSV Exports:** Filter-aware CSV downloads for users (`/api/admin/export/users`), reports (`/api/admin/export/items`), and activity logs (`/api/admin/export/activity`).

---

## Instructor Test Accounts & Shared Group (Demo Data)

> **Security Note:** Per course rules, test account passwords are **not** stored in this README or in the repository. Passwords are provided exclusively in the Final Submission Form and the PDF Report Appendix.

### Test Accounts (2 Admins & 3 Normal Users)

| Account Email (User ID) | Display Name | System Role | Purpose |
| :--- | :--- | :--- | :--- |
| `admin@campus.edu` | Campus Chief Admin | `admin` | Primary administrator account for dashboard, RBAC, audit log, and exports |
| `moderator@campus.edu` | Security Desk Moderator | `admin` | Secondary administrator account (also tests admin-cannot-ban-admin safety rule) |
| `student@campus.edu` | Sakif Ishmam | `user` | Normal user & **Owner** of the shared group *Main Library Lost & Found Team* |
| `alice@campus.edu` | Alice Rahman | `user` | Normal user & group member with **Create + Edit + Assign** permissions |
| `rafi@campus.edu` | Rafi Ahmed | `user` | Normal user & group member with **Create-Only** permission (cannot edit or assign) |

### Shared Group with Mixed Permissions

| Group Name | Member Email | Group Role | `can_create` | `can_edit` | `can_assign` | Owner Actions (Delete / Rename / Manage Members) |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: |
| **Main Library Lost & Found Team** | `student@campus.edu` | `owner` | Yes | Yes | Yes | **Allowed (Owner)** |
| **Main Library Lost & Found Team** | `alice@campus.edu` | `member` | Yes | Yes | Yes | Denied (`403`) |
| **Main Library Lost & Found Team** | `rafi@campus.edu` | `member` | Yes | No (`403`) | No (`403`) | Denied (`403`) |

---

## Project Architecture

```text
backend/
  main.py             # FastAPI app entrypoint, CORS, security headers, router registration
  models.py           # SQLModel database tables and Pydantic schemas
  database.py         # Database engine (PostgreSQL / SQLite) and session dependency
  security.py         # Pepper + Argon2 password hashing, HS256 JWT, session token hashing
  rbac.py             # Reusable RBAC permission constants, dependencies, and ownership/group guards
  crud.py             # Repository layer holding all SQLModel queries
  service.py          # Business logic, brute-force lockout, audit logging, CSV exports, demo seeding
  routers/
    items.py          # /api/items CRUD endpoints
    auth.py           # /api/auth register, login, refresh, logout, and device sessions
    profile.py        # /api/profile update, password change, and personal reports
    groups.py         # /api/groups shared recovery teams and member permissions
    admin.py          # /api/admin stats, user inspector, bulk actions, audit log, security, exports
  tests/
    test_auth_and_rbac.py  # Automated pytest suite for login, lockout, JWT rejection, and RBAC rules

frontend/src/
  App.tsx             # Root application state, session restoration, and navigation
  api.ts              # Fetch client with silent token refresh and ban/expiry interceptors
  types.ts            # TypeScript interfaces
  components/         # Header, FilterBar, ItemCard, ItemList, ReportModal, ItemDetailModal
  pages/              # LoginPage, SignupPage, ProfilePage, GroupsPage, AdminPage
```

---

## Running Locally & Executing Automated Tests

### 1. Backend & Pytest Suite
```bash
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
pytest
fastapi dev main.py
```

### 2. Frontend
```bash
cd frontend
npm install
npm run dev
```
