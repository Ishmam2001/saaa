# Product Requirements Document (PRD)
## Campus Lost & Found Management System

| **Field** | **Details** |
|---|---|
| Product | Campus Lost & Found Management System |
| Releases | MVP (Mid-term) and Beta (Final) |
| Status | Complete (Beta Released) |
| Related Documents | SRS, TDD |
| Frontend | React with TypeScript |
| Backend | FastAPI with Python |

---

## 1. Purpose

Campus Lost & Found is a web-based application designed to help campus members report lost and found items and easily access information about reported belongings.

The frontend is developed using **React with TypeScript**, while the backend uses **FastAPI with Python**.

The project is developed in two releases:

- **MVP (Mid-term):** A basic application that supports lost-and-found report management and runs locally.
- **Beta (Final):** An enhanced version that introduces JWT authentication, token refresh & revocation, account lockout protection, role-based permissions (user, moderator, admin), campus team desks, and cloud deployment on Vercel.

## 2. Problem Statement

Information about lost and found items is often shared through group chats, social media, or physical notice boards. This makes it difficult for students to find relevant information and keep track of reported items.

Campus Lost & Found solves this problem by providing a centralized platform where campus members can create, browse, view, update, and manage lost-and-found reports securely.

## 3. Project Goals

| Goal | Release |
|---|---|
| Create, view, update, and delete lost-and-found reports through REST APIs. | MVP |
| Run the frontend and backend locally as an integrated application. | MVP |
| Implement user authentication (JWT access & refresh tokens) and a profile dashboard. | Beta |
| Add brute-force protection (lockout after 5 failed login attempts for 15 minutes). | Beta |
| Introduce user, moderator, and admin roles with permission checks and ownership rules. | Beta |
| Add Campus Team Desks (Security Desk, Library Front Desk, IT Helpdesk) for staff. | Beta |
| Implement security scopes and token revocation on logout. | Beta |
| Deploy the application live on Vercel with PostgreSQL-ready storage. | Beta |

## 4. Target Users

| User | Description | Release |
|---|---|---|
| Campus Member | Can create, browse, and view lost-and-found reports without an account. | MVP |
| Registered Member (`user`) | Can sign in, manage profile, change password, and edit/delete own reports. | Beta |
| Desk Staff (`moderator`) | Can manage items assigned to campus desks (Security, Library, IT Helpdesk). | Beta |
| Administrator (`admin`) | Full system access: manage all reports, users, roles, and view analytics. | Beta |

### 4.1 User Stories

**MVP**

- As a campus member, I want to create a lost or found report so that others can find information about the item.
- As a campus member, I want to browse and view reports so that I can search for lost belongings.
- As a campus member, I want to update or delete a report so that I can correct or remove outdated information.

**Beta**

- As a member, I want to register and sign in securely so that I can access my personal account.
- As a member, I want my account protected against password guessing (temporary lockout after 5 failed attempts).
- As a member, I want my session to stay active smoothly via refresh tokens, and end securely when I log out.
- As a member, I want to view my profile and edit or delete only my own reports.
- As a desk moderator, I want to view and resolve reports assigned to campus desks (Security, Library, IT).
- As an administrator, I want administrative permissions to manage all reports, user roles, and team desks.
- As a member, I want the API to reject unauthorized requests with proper HTTP status codes (`401`, `403`).

## 5. Project Scope

### 5.1 MVP — Mid-term Release

The MVP focused on the core lost-and-found report management functionality.

**Included Features:**

- Create lost and found reports.
- List and browse available reports with search and category filters.
- View individual report details.
- Update existing reports.
- Delete reports.
- Validate report information and return clear API error messages.
- Integrate the React frontend with the FastAPI REST API.
- Run the application locally.

**Limitations:**

- No user registration or login.
- No role-based permissions or ownership restrictions.
- No authentication requirements.

### 5.2 Beta — Final Release

The Beta release extends the MVP by introducing authentication, authorization, team desks, and deployment.

**Included Features:**

- User registration, login, and logout.
- JWT authentication with access token (15 min) and rotating refresh token (7 days).
- Token revocation blacklist on logout.
- Account lockout protection: 5 consecutive failed login attempts locks account for 15 minutes.
- User profile dashboard and secure password change.
- Three user roles: `user`, `moderator`, and `admin`.
- Backend ownership checks: users can only edit/delete their own reports; staff/admin can manage all.
- Campus Team Desks (`Security Desk`, `Library Front Desk`, `IT Helpdesk`).
- Application deployment on Vercel with HTTPS and PostgreSQL support.

## 6. Functional Features and Priorities

| Feature | Release | Priority |
|---|---|---|
| Create lost or found reports | MVP | Must Have |
| List and view reports | MVP | Must Have |
| Update and delete reports | MVP | Must Have |
| Input validation and error handling | MVP | Must Have |
| User registration and login | Beta | Must Have |
| JWT access & rotating refresh tokens | Beta | Must Have |
| Token revocation on logout | Beta | Must Have |
| Account lockout protection (5 failed attempts) | Beta | Must Have |
| User profile dashboard & password change | Beta | Must Have |
| Role-based access control (`user`, `moderator`, `admin`) | Beta | Must Have |
| Ownership-based edit and delete permissions | Beta | Must Have |
| Campus team desks (Security, Library, IT) | Beta | Must Have |
| Application deployment on Vercel | Beta | Must Have |

## 7. Milestones and Deliverables

### 7.1 MVP — Mid-term

1. Complete project documentation (PRD, SRS, and TDD).
2. Design the report data model and configure the local SQLite database.
3. Implement and test report CRUD REST API endpoints.
4. Develop the React frontend with TypeScript.
5. Integrate the frontend with the FastAPI backend.
6. Test the core application functionality.
7. Demonstrate the application running locally.

### 7.2 Beta — Final

1. Implement user registration, login, and logout with JWT tokens.
2. Add access token expiration (15 min) and refresh token rotation (7 days).
3. Implement token revocation blacklist and account lockout protection.
4. Develop user profile dashboard and password change endpoint.
5. Implement `user`, `moderator`, and `admin` roles with backend RBAC dependencies.
6. Add item ownership checks and campus team desks.
7. Write and run automated test suite (`backend/tests/test_auth_and_rbac.py`).
8. Deploy the application to Vercel and verify live HTTPS endpoints.
9. Deliver final documentation, report, and project defense.

## 8. Success Criteria

### 8.1 MVP Success Criteria

The MVP was considered successful when:

- The application ran successfully in a local environment.
- Users could create, view, update, and delete reports through the frontend.
- All report CRUD operations worked correctly through the REST API.
- Invalid report information was rejected with appropriate error messages.
- The React frontend communicated successfully with the FastAPI backend.

### 8.2 Beta Success Criteria

The Beta release is considered successful when:

- Users can register, log in, refresh tokens, and log out securely.
- Repeated failed logins (5 attempts) lock the account for 15 minutes.
- Revoked tokens cannot be reused after logout.
- Regular users cannot edit or delete reports created by other members.
- Regular users cannot access admin or moderator endpoints.
- Administrators and moderators can manage reports and team desk items.
- All automated security and RBAC tests pass (100%).
- The application is deployed and accessible live over HTTPS on Vercel.

## 9. Assumptions and Constraints

- The application is designed primarily for a single campus community.
- Passwords are encrypted using modern hashing (Argon2 / PBKDF2) and never stored in plain text.
- Token secrets are loaded from environment variables and never hardcoded in repository files.
- Local development uses SQLite; production can use SQLite or hosted PostgreSQL via `DATABASE_URL`.
- Deployment is configured for Vercel with serverless Python backend functions.

---
