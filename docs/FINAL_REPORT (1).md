Independent University, Bangladesh CSE309 Web Application & Internet 

# **Final Project Submission Report** 

**Submitted by:** Name: Sakif Ishmam Student ID: 2220681 Section: Section 6 Team Number: Individual (1) 

Project Title: Campus Lost & Found Management System Vercel URL: https://campus-lost-found-management-system.vercel.app GitHub Repository: https://github.com/Ishmam2222/Campus-Lost-Found-Management-System 

**Submitted To:** 

Sayef Reyadh Adjunct Lecturer Department of Computer Science & Engineering Independent University, Bangladesh 

Semester: Autumn 2026 Date of Submission: 26 November 2026 

## **Table of Contents** 

- <u>1. Executive Summary</u> 

- <u>Part A. Product Requirements (PRD)</u> 

- <u>Part B. Software Requirements (SRS)</u> 

- <u>Part C. Technical Design (TDD)</u> 

- <u>Part D. Evidence: Screenshots of the Running Application</u> 

- <u>Part E. Project Management and Conclusion</u> 

- <u>Appendix. Test Accounts and Video Demo</u> 

### **List of Figures and Screenshots** 

- **Figure 1:** Use Case Diagram ( `Part B5` ) 

- **Figure 2:** System Architecture ( `Part C2` ) 

- **Figure 3:** Database ER Diagram ( `Part C4` ) 

- **Figure 4:** JWT Login, Refresh & Logout Flow ( `Part C6` ) 

- **Figure 5:** Request Authorization Flow ( `Part C7` ) 

- **Figure 6:** Admin Dashboard Pages & Endpoints ( `Part C8` ) 

- **Figure 7:** Vercel Deployment Layout ( `Part C10` ) 

- **Screenshots S1–S21:** Live Application Evidence ( `Part D` ) 

## **1. Executive Summary** 

### **1.1 Project overview** 

The **Campus Lost & Found Management System** is a web app for university students, staff, and security desks to report and recover lost or found items on campus. Signed-in users manage their own reports, work together in shared recovery groups (like the Library Desk), and track items from `active` to `claimed` and `resolved` . 

**Stack:** React with TypeScript (Vite), FastAPI with SQLModel, PostgreSQL / SQLite database, deployed on Vercel. 

### **1.2 What was built since the mid assessment** 

- **JWT authentication with refresh and logout:** Added signup, login, Pepper + Argon2 password hashing, 15-minute JWT access tokens, `HttpOnly` refresh cookies with token rotation, session revocation on logout, and an active sessions list on the Profile page. 

- **Role based access control (roles, ownership, group permissions):** Added a single `backend/rbac.py` module enforcing `user` and `admin` roles, strict personal report ownership (users only see and edit their own reports), and shared **Recovery Teams** with per-member `can_create` , `can_edit` , and `can_assign` permissions. 

- **Admin dashboard (users, content, activity log, security view, exports):** Built a 4-tab Admin Panel showing 7-day stats, user search/filters, user detail inspection, ban/unban, role changes, bulk delete, activity log, failed login monitor, and CSV downloads. 

- **Audit logging, brute-force protection, deployment on Vercel:** Added automatic activity logging (user, IP, browser, time), a 15-minute account lockout after 5 wrong passwords ( `429` ), automated `pytest` tests, and live Vercel deployment. 

### **1.3 Live links** 

**Table 1: Live Links** 

|**Resource**|**URL**|
|---|---|
|**GitHub Repository**|https://github.com/Ishmam2222/Campus-Lost-<br>Found-Management-System|
|**Vercel Frontend**|https://campus-lost-found-management-<br>system.vercel.app|
|**Vercel API**|https://campus-lost-found-management-<br>system.vercel.app/api/items|
|**Vercel API Docs**|https://campus-lost-found-management-<br>system.vercel.app/api/docs|



|**Resource**|**URL**|
|---|---|
|**Video Demo**|_(Provided in Final Submission Form and PDF_<br>_Appendix)_|



## **Part A. Product Requirements (PRD)** 

### **A1. Problem Statement** 

Students and staff regularly lose ID cards, calculators, keys, chargers, and notebooks in classrooms, labs, and the library. Today, people post in big Facebook or WhatsApp groups or ask guards at building gates. Those chat posts quickly get buried, have no search or category filter, and nobody can tell if an item was already returned. 

Also, campus lost-and-found desks are run by teams of student volunteers across different shifts. Without proper login, ownership rules, and team permissions, anyone could edit or delete another student's report, post spam, or mess up desk records with no way to track who did it. 

### **A2. Users and Roles** 

**Table 2: Users and Roles** 

|**Role**|**Who They Are**|**Main Tasks**|**What They Must**<br>**Never Be Able To Do**|
|---|---|---|---|
|**Visitor**|Unauthenticated guest|Open the Login or Sign<br>Up page|View, create, edit, or<br>delete any report or call<br>protected APIs (`401`)|
|**User (****`user`)**|Student or staff<br>member|Create reports,<br>view/edit/delete their<br>own reports, update<br>profile, change<br>password, manage<br>sessions, create/join<br>groups|Read, edit, or delete<br>another user's personal<br>report (`403`) or open<br>`/api/admin/*`<br>(`403`)|
|**Group Member**|User added to a shared<br>Recovery Team|View group reports and<br>use only the<br>permissions granted to<br>them (`can_create`,<br>`can_edit`,<br>`can_assign`)|Perform ungranted<br>group actions (`403`),<br>delete group items<br>(`403`), or manage<br>members/group settings<br>(`403`)|
|**Group Owner**|Creator of a Recovery<br>Team|Full group control:<br>create, edit, assign,<br>delete group items,<br>rename/delete group,<br>and add/remove<br>members|Access system-wide<br>`/api/admin/*`<br>routes or other users'<br>private reports (`403`)|
|**Admin (****`admin`)**|Campus security|View stats,|Change their own role|



|**Role**|**Who They Are**|**Main Tasks**|**What They Must**<br>**Never Be Able To Do**|
|---|---|---|---|
||moderator|search/inspect users,<br>change roles,<br>ban/unban users, delete<br>spam reports/groups,<br>view audit & security<br>logs, export CSVs|(`400`) or ban<br>themselves or another<br>admin (`400`)|



### **A3. Goals and Success Measures** 

**Table 3: Goals and Measures** 

|**Goal ID**|**Goal**|**Testable Measure**|
|---|---|---|
|**G-01**|Fast lost and found reporting<br>and filtering|All report CRUD and filter<br>requests finish in`< 500 ms`<br>with`200`/`201`status|
|**G-02**|Short-lived, revocable JWT<br>sessions|Access tokens expire in`<= 15`<br>`minutes`; expired, tampered,<br>or logged-out tokens return<br>`401`|
|**G-03**|No user can read or change<br>another user's data|`100%`of requests to read, edit,<br>or delete another user's personal<br>report return`403`|
|**G-04**|Fine-grained team permissions<br>work on the server|Members without`can_edit`<br>or`can_assign`get`403`;<br>non-owners get`403`on<br>delete/rename/member changes|
|**G-05**|Brute-force protection and<br>instant ban blocking|5 wrong passwords lock the<br>account for`15 minutes`<br>(`429`); banned users get`403`<br>on their very next request|



### **A4. Scope and Non-Goals** 

####  **In Scope:** 

- Lost & found report CRUD, search, category/type/status filters, and status updates ( `active` , `claimed` , `resolved` ). 

- Signup, login, logout, Pepper + Argon2 password hashing, 15-min JWT + 7-day `HttpOnly` refresh cookie rotation, device session revoke, profile update, password change, and 5-attempt lockout. 

- `backend/rbac.py` module, strict personal ownership, shared Recovery Teams ( `/api/groups` ) with `can_create` , `can_edit` , `can_assign` , and frontend `ProtectedRoute` . 

   - Admin dashboard (stats, user list & detail modal, ban/unban, role change, bulk actions, content moderation, activity log, security view, CSV exports) and Vercel deployment. 

- **Out of Scope:** 

   - _Image uploads:_ Skipped to avoid external cloud storage setup and keep the app fast. 

   - _Email/SMS notifications:_ Skipped so demo testing works without external SMTP servers. 

- **Assumptions and Constraints:** 

   - Runs on modern browsers with cookies enabled over HTTPS on Vercel serverless functions backed by a hosted database. 

### **A5. User Stories** 

**Table 4: User Stories** 

|**ID**|**Role**|**User Story**|
|---|---|---|
|**US-01**|Visitor|As a visitor, I want to sign up<br>and log in with my campus<br>email so that I can report lost or<br>found items.|
|**US-02**|User|As a logged-in user, I want my<br>session to refresh automatically<br>and let me revoke old device<br>logins so that my account stays<br>safe.|
|**US-03**|User|As a logged-in user, I want to<br>update my name and change my<br>password after entering my<br>current password so that I can<br>keep my account secure.|
|**US-04**|User|As a report owner, I want only<br>my own reports shown to me<br>and other users blocked from<br>editing or deleting them so that<br>my data stays private.|
|**US-05**|Group Owner|As a group owner, I want to<br>create a recovery team, add<br>members by email, and set each<br>member's`create`,`edit`, and<br>`assign`permissions so that<br>volunteers have appropriate<br>access.|
|**US-06**|Group Member|As a group member with<br>`can_create`only, I want to<br>log found items into the team<br>desk while being blocked from<br>editing or deleting existing|



|**ID**|**Role**|**User Story**|
|---|---|---|
|||items.|
|**US-07**|Group Member|As a group member with<br>`can_edit`and<br>`can_assign`, I want to<br>update item status and assign a<br>handler so the team knows who<br>is handling a return.|
|**US-08**|Admin|As an admin, I want to view 7-<br>day overview stats, filter users,<br>and inspect a user's reports,<br>groups, sessions, and recent<br>activity in one place.|
|**US-09**|Admin|As an admin, I want to ban or<br>unban users (with a reason) and<br>delete inappropriate reports in<br>bulk so that bad actors are<br>stopped immediately.|
|**US-10**|Admin|As an admin, I want to filter the<br>activity log, check failed logins<br>by IP and account, and export<br>users, reports, and logs to CSV.|



## **Part B. Software Requirements (SRS)** 

### **B1. Introduction** 

- **Purpose:** Defines the functional and non-functional requirements, permission rules, use cases, and tests for the Campus Lost & Found Management System. 

- **Intended Readers:** Course instructor, teaching assistants, and developers. 

|**Term**|**Meaning**|
|---|---|
|**JWT**|Signed JSON Web Token carrying user claims<br>(`sub`,`email`,`role`,`iat`,`exp`).|
|**Access Token**|Short-lived JWT (`15 minutes`) sent in the<br>`Authorization: Bearer`header.|
|**Refresh Token**|7-day random token stored in an`HttpOnly`<br>cookie and hashed (SHA-256) in<br>`campus_session`.|
|**Pepper**|Server-side secret (`PASSWORD_PEPPER`) from<br>environment variables mixed into passwords<br>before Argon2 hashing.|
|**RBAC**|Role-Based Access Control enforcing role,|



|**Term**|**Meaning**|
|---|---|
||ownership, and group permissions on the server.|
|**Audit Log**|Database table (`campus_activity_log`)|
||recording security and user actions with IP,<br>browser, and time.|



 **References:** RFC 7519 (JWT), OWASP ASVS 4.0, FastAPI & SQLModel documentation. 

### **B2. Functional Requirements** 

**Table 5: Authentication & JWT Requirements (** **`FR-AUTH` )** 

|**ID**|**Requirement**|
|---|---|
|**FR-AUTH-01**|The system shall allow visitors to register<br>(`POST /api/auth/register`) with name,<br>email, and password (`>= 6`chars) with default<br>role`user`(`409`if email exists).|
|**FR-AUTH-02**|The system shall mix passwords with<br>`PASSWORD_PEPPER`(HMAC-SHA256) and<br>hash them with Argon2, never storing plain text.|
|**FR-AUTH-03**|The system shall issue a signed HS256 JWT<br>access token (`<= 15 minutes`) containing<br>`sub`,`email`,`role`,`iat`, and`exp`on login.|
|**FR-AUTH-04**|The system shall verify the JWT on every<br>protected request and return`401`if it is missing,<br>malformed, tampered, or expired.|
|**FR-AUTH-05**|The system shall store a SHA-256 hash of the<br>`HttpOnly`refresh cookie in<br>`campus_session`, rotate it on`POST`<br>`/api/auth/refresh`, and revoke it on<br>`POST /api/auth/logout`.|
|**FR-AUTH-06**|The system shall allow users to list their active<br>sessions (`GET /api/auth/sessions`) and<br>revoke single, other, or all sessions.|
|**FR-AUTH-07**|The system shall lock an account for 15 minutes<br>(`429 Too Many Requests`) after 5<br>consecutive failed login attempts.|
|**FR-AUTH-08**|The system shall allow users to update their<br>display name (`PUT /api/profile`) and<br>change their password (`POST`<br>`/api/profile/password`) after verifying<br>their current password.|



**Table 6: RBAC & Group Requirements (** **`FR-RBAC` )** 

|**ID**|**Requirement**|
|---|---|
|**FR-RBAC-01**|The system shall store user roles (`user`,<br>`admin`) in the database and enforce permissions<br>on the server using one reusable module<br>(`backend/rbac.py`).|
|**FR-RBAC-02**|The system shall allow normal users to read,<br>edit, or delete only their own personal reports<br>(`owner_id == user.id`), returning`403`<br>for other users' reports.|
|**FR-RBAC-03**|The system shall allow users to create shared<br>Recovery Groups (`POST /api/groups`)<br>where the creator becomes the group`owner`.|
|**FR-RBAC-04**|The system shall check group member flags<br>(`can_create`,`can_edit`,`can_assign`)<br>and return`403`when a member tries an action<br>they were not granted.|
|**FR-RBAC-05**|The system shall allow only the group`owner`<br>(or system`admin`) to delete group reports,<br>rename/delete the group, or add/remove/edit<br>group members (`403`otherwise).|
|**FR-RBAC-06**|The system shall block an admin from changing<br>their own role (`400`) or banning themselves or<br>another admin (`400`).|
|**FR-RBAC-07**|The system shall check`user.is_banned`on<br>every authenticated request and reject a banned<br>user on their very next request with`403`.|



**Table 7: Admin Dashboard & Audit Requirements (** **`FR-ADM` )** 

|**ID**|**Requirement**|
|---|---|
|**FR-ADM-01**|The system shall provide overview counts (`GET`<br>`/api/admin/stats`) including users,<br>admins, banned users, reports, groups, and new<br>users/reports in the last 7 days.|
|**FR-ADM-02**|The system shall provide a paginated user list<br>(`GET /api/admin/users`) with search and<br>role/status filters, plus a user detail view<br>(`GET /api/admin/users/{id}`) with<br>reports, groups, sessions, and activity.|
|**FR-ADM-03**|The system shall allow admins to ban/unban<br>users with a reason, change roles, revoke<br>sessions, and run bulk ban/unban actions.|
|**FR-ADM-04**|The system shall allow admins to list and filter<br>all reports and groups, delete single<br>items/groups, and bulk-delete reports.|



|**ID**|**Requirement**|
|---|---|
|**FR-ADM-05**|The system shall log login, failed login, logout,<br>password change, role change, ban, delete,<br>export, token rejection, and RBAC denial events<br>with user, IP, browser, and time (`GET`<br>`/api/admin/activity`).|
|**FR-ADM-06**|The system shall provide a security summary of<br>failed logins by IP and account (`GET`<br>`/api/admin/security`) and filter-aware<br>CSV exports<br>(`/api/admin/export/users`,`items`,<br>`activity`).|



**Table 8: Lost & Found Domain Requirements (** **`FR-DOMAIN` )** 

|**ID**|**Requirement**|
|---|---|
|**FR-DOMAIN-01**|The system shall allow authenticated users to<br>create lost or found reports (`POST`<br>`/api/items`) with type, title, category,<br>location, description, contact info, and status.|
|**FR-DOMAIN-02**|The system shall list the user's reports (`GET`<br>`/api/items`) with type, category, status, and<br>keyword filters, and return single report details<br>(`GET /api/items/{id}`).|
|**FR-DOMAIN-03**|The system shall allow authorized owners,<br>permitted group members, and admins to update<br>report fields/status (`PUT`<br>`/api/items/{id}`) and delete reports<br>(`DELETE /api/items/{id}`).|



### **B3. Non-Functional Requirements** 

**Table 9: Non-Functional Requirements** 

|**ID**|**Area**|**Requirement**|
|---|---|---|
|**NFR-01**|Security|Secrets (`JWT_SECRET`,<br>`PASSWORD_PEPPER`,<br>`DATABASE_URL`) come from<br>environment variables (only<br>`.env.example`in git); failed<br>logins return a generic<br>`"Invalid email or`<br>`password."`message.|
|**NFR-02**|Security|Every response sets`X-`<br>`Content-Type-Options:`<br>`nosniff`,`X-Frame-`|



|**ID**|**Area**|**Requirement**|
|---|---|---|
|||`Options: DENY`, and hides<br>`500`stack traces.|
|**NFR-03**|Performance|User pagination is capped at<br>`limit <= 100`(default<br>`10`/`20`) and API calls respond<br>in`< 500 ms`.|
|**NFR-04**|Usability|Protected pages use<br>`ProtectedRoute.tsx`with<br>no guest-content flash while<br>restoring sessions, plus<br>responsive loading, error, and<br>empty states.|
|**NFR-05**|Reliability|Production data persists in a<br>hosted database on Vercel<br>(`DATABASE_URL`) across<br>restarts.|
|**NFR-06**|Maintainability|All RBAC checks live in<br>`backend/rbac.py`and 7<br>automated`pytest`tests verify<br>auth, lockout, token rejection,<br>and RBAC.|



### **B4. Roles and Permission Matrix** 

**Table 10: Endpoint Groups by Role** 

|**Endpoint**<br>**Group**|**Visitor**|**User**|**Group**<br>**Member**|**Group Owner**|**Admin**|
|---|---|---|---|---|---|
|`GET`<br>`/api/healt`<br>`h`,`POST`<br>`/api/auth/`<br>`register`,<br>`POST`<br>`/api/auth/`<br>`login`|Yes|Yes|Yes|Yes|Yes|
|`GET/PUT/`<br>`DELETE`<br>`/api/items`<br>`/*`(Personal<br>Reports)|No (`401`)|**Own**(`403`<br>others)|**Own**|**Own**|Yes (All)|
|`/api/`<br>`profile`,<br>`/api/profi`<br>`le/passwor`<br>`d`,|No (`401`)|**Own**|**Own**|**Own**|**Own**|



|**Endpoint**<br>**Group**|**Visitor**|**User**|**Group**<br>**Member**|**Group Owner**|**Admin**|
|---|---|---|---|---|---|
|`/api/auth/`<br>`sessions`||||||
|`GET/`<br>`POST`<br>`/api/group`<br>`s`|No (`401`)|Yes|Yes|Yes|Yes|
|`POST`<br>`/api/group`<br>`s/{id}/ite`<br>`ms`,`PUT`<br>`/api/items`<br>`/{id}`(Group<br>Item)|No (`401`)|No (`403`)|**If flag granted**|Yes|Yes|
|`DELETE`<br>Group Item,<br>`PATCH/DELE`<br>`TE`<br>`/api/group`<br>`s/{id}`,<br>Manage<br>Members|No (`401`)|No (`403`)|No (`403`)|**Yes**|Yes|
|`/api/`<br>`admin/*`<br>(Stats, Users,<br>Moderation,<br>Activity,<br>Security,<br>Exports)|No (`401`)|No (`403`)|No (`403`)|No (`403`)|**Yes**|



**Table 11: Group Permissions** 

|**Group Role /**<br>**Flag**|**Create Group**<br>**Item**|**Edit Group**<br>**Item & Status**|**Assign**<br>**Member**|**Delete Group**<br>**Item**|**Rename/**<br>**Delete Group**<br>**& Manage**<br>**Members**|
|---|---|---|---|---|---|
|**`can_create`**<br>**`= True`**|<br>Yes (`201`)|No (`403`)|No (`403`)|No (`403`)|No (`403`)|
|**`can_edit =`**<br>**`True`**|<br>If<br>`can_create`|Yes (`200`)|If<br>`can_assign`|No (`403`)|No (`403`)|
|**`can_assign`**<br>**`= True`**|<br>If<br>`can_create`|If`can_edit`|Yes (`200`)|No (`403`)|No (`403`)|
|**Group****`owner`**|Yes (`201`)|Yes (`200`)|Yes (`200`)|**Yes (****`200`)**|**Yes (****`200`)**|



#### **Table 12: Admin Rules** 

|**Rule**|**Server Behavior**|**S**|**tatus Code**|
|---|---|---|---|
|**Cannot change own role**|`PATCH`<br>`/api/admin/users/{sel`<br>`f_id}/role`is blocked|`400 Bad`|`Request`|
|**Cannot ban self or another**<br>**admin**|`POST`<br>`/api/admin/users/{adm`<br>`in_id}/ban`is blocked (and<br>skipped in`bulk-ban`)|`400 Bad`|`Request`|
|**Admin group moderation**|System admins can inspect or<br>delete any group or group<br>report for moderation|`200 OK`||



### **B5. Use Cases** 

**Figure 1: Use Case Diagram** 

```
flowchart LR
    Visitor(["Visitor"])
    User(["User"])
    Member(["Group Member"])
    Owner(["Group Owner"])
    Admin(["Admin"])
    UC1(["UC-01: Log In & Refresh Token"])
    UC2(["UC-02: Create Lost/Found Report"])
    UC3(["UC-03: Change Group Member Permissions"])
    UC4(["UC-04: Denied Action (403 RBAC)"])
    UC5(["UC-05: Ban User & Revoke Sessions"])
    UC6(["UC-06: View Audit Log & Export CSV"])
    Visitor --> UC1
    User --> UC1
    User --> UC2
    User --> UC4
    Member --> UC4
    Owner --> UC3
    Admin --> UC5
    Admin --> UC6
```

**Table 13: Use Case Descriptions** 

|**ID & Name**|**Actor**|**Preconditions**|**Main Flow**|**Alternative /**<br>**Error Flows**|**Postconditions**|
|---|---|---|---|---|---|
|**UC-01: Log In**<br>**& Refresh**|User|Account exists,<br>not banned or<br>locked|1. Submit<br>email &<br>password to<br>`POST`<br>`/api/auth/`<br>`login`.2.<br>Server checks<br>peppered<br>Argon2 hash<br>and logs|- Wrong<br>password:`401`<br>+ increment<br>fail count.- 5th<br>wrong try:<br>`429`(15m<br>lock).- Banned<br>user:`403`.|<br>User has a<br>valid 15-min<br>JWT and<br>active session<br>row.|



|**ID & Name**|**Actor**|**Preconditions**|**Main Flow**|**Alternative /**<br>**Error Flows**|**Postconditions**|
|---|---|---|---|---|---|
||||`login`.3.<br>Returns 15-<br>min JWT +<br>sets<br>`HttpOnly`<br>refresh<br>cookie.4. On<br>`401`, client<br>calls<br>`POST`<br>`/api/auth/`<br>`refresh`to<br>rotate cookie<br>and get a new<br>JWT.|||
|**UC-02:**<br>**Create a**<br>**Report**|User|Logged in with<br>valid JWT|<br>1. Fill report<br>form and<br>submit<br>`POST`<br>`/api/items`<br>.2. Server<br>validates<br>fields, attaches<br>`owner_id`<br>and<br>`owner_name`<br>, saves item,<br>and logs<br>`item_creat`<br>`e`(`201`).|- Missing<br>title/location:<br>`422`.-<br>Missing/expire<br>d token:`401`.|Report is saved<br>and shown in<br>the user's<br>report list.|
|**UC-03:**<br>**Change**<br>**Member**<br>**Permissions**|Group Owner|Logged in as<br>group`owner`|1. Toggle<br>`can_edit`/<br>`can_assign`<br>for a member<br>on Recovery<br>Teams page.2.<br>Send<br>`PATCH`<br>`/api/group`<br>`s/{id}/mem`<br>`bers/{uid}`<br>.3. Server<br>checks<br>`owner_only`<br>`=True`,<br>updates flags,<br>and logs<br>`group_perm`|<br>- Caller is not<br>group owner:<br>`403`.-<br>Group/member<br>missing:`404`.|Member's<br>permissions<br>are updated in<br>`campus_gro`<br>`up_member`.|



|**ID & Name**|**Actor**|**Preconditions**|**Main Flow**|**Alternative /**<br>**Error Flows**|**Postconditions**|
|---|---|---|---|---|---|
||||`ission_upd`<br>`ate`(`200`).|||
|**UC-04:**<br>**Denied Action**<br>**by RBAC**|User / Member|Logged in<br>without<br>required<br>permission|1. User tries to<br>edit another<br>user's item,<br>edit a group<br>item without<br>`can_edit`, or<br>call<br>`/api/admin`<br>`/stats`.2.<br>`rbac.py`<br>blocks the<br>request, logs<br>`rbac_denie`<br>`d`, and returns<br>`403`.|<br>- No token or<br>expired token:<br>`401`<br>(`token_rej`<br>`ected`).|Action is<br>blocked and<br>recorded in the<br>audit log.|
|**UC-05: Ban a**<br>**User**|Admin|Logged in as<br>`admin`|1. Click Ban<br>on a user and<br>enter a reason<br>(`POST`<br>`/api/admin`<br>`/users/{id`<br>`}/ban`).2.<br>Server checks<br>target is not an<br>admin, sets<br>`is_banned`<br>`= True`,<br>revokes all<br>user sessions,<br>and logs<br>`user_ban`<br>(`200`).|- Target is self<br>or another<br>admin:`400`.-<br>Caller is not<br>admin:`403`.|User is banned<br>and blocked on<br>their very next<br>request (`403`).|
|**UC-06: Read**<br>**Audit Log &**<br>**Export**|Admin|Logged in as<br>`admin`|1. Open<br>Activity Audit<br>Log tab and<br>filter by event<br>or keyword<br>(`GET`<br>`/api/admin`<br>`/activity`).<br>2. Click Export<br>CSV (`GET`<br>`/api/admin`<br>`/export/ac`<br>`tivity`),|<br>- Caller is not<br>admin:`403`.|Filtered logs<br>are viewed and<br>downloaded as<br>CSV.|



|**ID & Name**|**Actor**|**Preconditions**|**Main Flow**<br>which logs<br>`data_expor`<br>`t`and<br>downloads the<br>CSV.|**Alternative /**<br>**Error Flows**|**Postconditions**|
|---|---|---|---|---|---|



### **B6. Acceptance Criteria** 

#### **Table 14: Acceptance Criteria** 

|**ID**|**Requirement**|**Given**|**When**|**Then**|
|---|---|---|---|---|
|**AC-01**|`FR-AUTH-04`|A JWT access<br>token has passed<br>its 15-minute`exp`<br>time|<br>Client calls<br>`GET`<br>`/api/profile`<br>with that token|Server returns<br>`401`<br>`Unauthorized`<br>and logs<br>`token_reject`<br>`ed`.|
|**AC-02**|`FR-AUTH-04`|A JWT access<br>token's signature<br>is modified|Client sends the<br>tampered token to<br>any protected<br>route|Server returns<br>`401`<br>`Unauthorized`<br>and logs<br>`token_reject`<br>`ed`.|
|**AC-03**|`FR-RBAC-02`|`student@camp`<br>`us.edu`owns<br>report`#1`and<br>`ammar@campus`<br>`.edu`is logged in|Ammar sends<br>`GET`,`PUT`, or<br>`DELETE`<br>`/api/items/1`|Server returns<br>`403`<br>`Forbidden`and<br>logs<br>`rbac_denied`.|
|**AC-04**|`FR-RBAC-04`|`sajid@campus`<br>`.edu`has<br>`can_create=T`<br>`rue,`<br>`can_edit=Fal`<br>`se`in the group|Sajid sends<br>`PUT`<br>`/api/items/{`<br>`group_item_i`<br>`d}`|Server returns<br>`403`<br>`Forbidden`.|
|**AC-05**|`FR-RBAC-07`|`sajid@campus`<br>`.edu`holds a<br>non-expired JWT<br>and an admin bans<br>him|<br>Sajid makes his<br>very next request<br>(`GET`<br>`/api/profile`)|Server checks<br>`user.is_bann`<br>`ed`and returns<br>`403`<br>`Forbidden`<br>`("Account`<br>`banned")`.|
|**AC-06**|`FR-RBAC-06`|`admin@campus`<br>`.edu`is logged in|Admin tries to ban<br>`moderator@ca`<br>`mpus.edu`or<br>change their own|Server returns<br>`400 Bad`<br>`Request`.|



|**ID**|**Requirement**|**Given**|**When**|**Then**|
|---|---|---|---|---|
||||role||
|**AC-07**|`FR-AUTH-07`|A user account<br>exists|Client sends 5<br>wrong password<br>attempts in a row|The 5th attempt<br>returns`429 Too`<br>`Many`<br>`Requests`and<br>locks the account<br>for 15 minutes.|



### **B7. Traceability Matrix** 

**Table 15: Traceability Matrix** 

|**Requirement ID**|**Endpoint / Frontend**<br>**Screen**|**Automated Test**<br>**(****`test_auth_and_r`**<br>**`bac.py`)**|**Screenshot**|
|---|---|---|---|
|**FR-AUTH-01, 02**|`POST`<br>`/api/auth/regis`<br>`ter`/<br>`SignupPage.tsx`|`test_brute_forc`<br>`e_login_lockout`,<br>`test_login_and_`<br>`refresh_flow`|**S1**,**S19**|
|**FR-AUTH-03, 04**|`POST`<br>`/api/auth/login`,<br>`security.py`/<br>`LoginPage.tsx`|`test_login_and_`<br>`refresh_flow`,<br>`test_token_reje`<br>`ction_missing_t`<br>`ampered_and_exp`<br>`ired`|**S1**,**S7**,**S21**|
|**FR-AUTH-05, 06**|`POST`<br>`/api/auth/refre`<br>`sh`,`/logout`,<br>`/sessions`/<br>`ProfilePage.tsx`|`test_login_and_`<br>`refresh_flow`|**S4**,**S8**,**S12**|
|**FR-AUTH-07**|`service.authent`<br>`icate_user`lockout|`test_brute_forc`<br>`e_login_lockout`|**S16**,**S21**|
|**FR-AUTH-08**|`PUT`<br>`/api/profile`,<br>`POST`<br>`/api/profile/pa`<br>`ssword`/<br>`ProfilePage.tsx`|Verified via Profile<br>API & UI|**S3**|
|**FR-RBAC-01**|`backend/rbac.py`<br>(`require_permiss`<br>`ion`)|`test_rbac_rule_`<br>`1_normal_user_d`<br>`enied_admin_end`<br>`points`|**S6**,**S21**|
|**FR-RBAC-02**|`backend/rbac.py`<br>(`enforce_item_ow`<br>`nership`)|`test_rbac_rule_`<br>`2_ownership_pre`<br>`vents_modifying`<br>`_other_users_re`|**S2**,**S6**,**S21**|



|**Requirement ID**|**Endpoint / Frontend**<br>**Screen**|**Automated Test**<br>**(****`test_auth_and_r`**<br>**`bac.py`)**|**Screenshot**|
|---|---|---|---|
|||`cord`||
|**FR-RBAC-03, 04, 05**|`/api/groups/*`,<br>`enforce_group_p`<br>`ermission`/<br>`GroupsPage.tsx`|`test_rbac_rule_`<br>`3_fine_grained_`<br>`group_permissio`<br>`ns`|**S5**,**S6**,**S21**|
|**FR-RBAC-06, 07**|`service.py`safety<br>checks,<br>`security.py`ban<br>check|`test_rbac_rule_`<br>`4_admin_safety_`<br>`and_immediate_b`<br>`an_enforcement`|**S9**,**S13**,**S21**|
|**FR-ADM-01, 02, 03**|`/api/admin/`<br>`stats`,`/users`,<br>`/ban`/<br>`AdminPage.tsx`|`test_rbac_rule_`<br>`1_...`,<br>`test_rbac_rule_`<br>`4_...`|**S10–S13**|
|**FR-ADM-04, 05, 06**|`/api/admin/`<br>`items`,`/activity`,<br>`/security`,<br>`/export/*`|`test_rbac_rule_`<br>`1_normal_user_d`<br>`enied_admin_end`<br>`points`|**S14–S17**|
|**FR-DOMAIN-01–03**|`/api/items/*`/<br>`App.tsx`,<br>`ItemList.tsx`|`test_rbac_rule_`<br>`2_ownership_pre`<br>`vents_modifying`<br>`_other_users_re`<br>`cord`|**S2**,**S20**|



## **Part C. Technical Design (TDD)** 

### **C1. Technology Stack** 

#### **Table 16: Technology Stack** 

|**Component**|**Technology**|**Version**|**Why We Chose It**|
|---|---|---|---|
|**Frontend**|React + TypeScript<br>(Vite)|React 18 / TS 5|Type safety for API<br>data and fast Vite<br>builds|
|**UI Styling**|Custom CSS3|CSS Grid/Flex|Clean responsive dark<br>UI with zero extra<br>library bloat|
|**Backend**|FastAPI (Python)|`>=0.141`(Py 3.12)|Built-in`/api/docs`,<br>Pydantic validation,<br>and`Depends()`for<br>RBAC|
|**ORM**|SQLModel|`>=0.0.22`|Combines|



|**Component**|**Technology**|**Version**|**Why We Chose It**|
|---|---|---|---|
||||SQLAlchemy tables<br>and Pydantic schemas<br>in one file|
|**Database**|PostgreSQL / SQLite|Postgres 15 / SQLite 3|Hosted PostgreSQL for<br>Vercel persistence;<br>SQLite for local dev|
|**Password Hashing**|`pwdlib[argon2]`+<br>HMAC Pepper|`>=0.2`|Argon2 hashing<br>combined with a<br>server-side<br>`PASSWORD_PEPPER`<br>secret|
|**JWT Library**|PyJWT|`>=2.8`|Standard HS256 JWT<br>signing and expiry<br>verification|
|**Testing**|Pytest + FastAPI<br>`TestClient`|`>=8.0`|Fast automated testing<br>of login, lockout, token<br>tampering, and RBAC|
|**Hosting**|Vercel|Serverless|Hosts both the React<br>SPA and FastAPI<br>`/api/*`under one<br>HTTPS domain|



### **C2. System Architecture** 

#### **Figure 2: System Architecture** 

```
flowchart LR
    Browser["Browser (React + TS)"]
    FE["Vercel Frontend (Static SPA)"]
    API["Vercel FastAPI (/api/*)"]
    DB[("Hosted Database")]
    Audit[("campus_activity_log")]
    Browser -->|"1. Load SPA"| FE
    Browser -->|"2. API Request + Authorization: Bearer <JWT>"| API
    Browser -->|"3. POST /api/auth/refresh (HttpOnly Cookie)"| API
    API -->|"4. Parameterized SQLModel Queries"| DB
    API -->|"5. Record Audit Events (IP, UA, Action)"| Audit
```

- **Arrows 1–2:** The browser loads the React app from Vercel and sends protected API calls to `/api/*` with `Authorization: Bearer <access_token>` . 

- **Arrows 3–5:** When the 15-minute JWT expires, the browser sends the `HttpOnly refresh_token` cookie to `/api/auth/refresh` , while FastAPI queries the database and writes every security and user event to `campus_activity_log` . 

### **C3. Backend and Frontend Structure** 

**Backend Folder Tree:** 

|`backend/`||
|---|---|
|`main.py`|`# FastAPI app, CORS, security headers, router`|
|`registration`||
|`models.py`|`# 6 SQLModel tables and Pydantic request/response`|
|`schemas`||
|`database.py`|`# Engine setup (PostgreSQL NullPool / SQLite) and`|
|`session dependency`||
|`security.py`<br>|`# Pepper + Argon2 hashing, HS256 JWT,`<br>|
|`get_current_user dependenc`|`y`|
|`rbac.py`|`# Reusable Permission constants, require_permission,`|
|`ownership & group checks`||
|`service.py`|`# Business logic, lockout, token rotation, CSV`|
|`exports, demo seeding`||
|`crud.py`<br>`routers/`|`# SQLModel database queries`|
|`auth.py`|`# /api/auth (register, login, refresh, logout,`|
|`sessions)`||
|`profile.py`|`# /api/profile (update name, change password)`|
|`items.py`|`# /api/items (lost & found report CRUD)`|
|`groups.py`|`# /api/groups (recovery teams & member permissions)`|
|`admin.py`|`# /api/admin (stats, users, bans, moderation, audit,`|
|`security, CSV)`||
|`tests/`||
|`test_auth_and_rbac.py`|`# 7 automated pytest tests`|



#### **Frontend Folder Tree:** 

|`frontend/src/`||
|---|---|
|`main.tsx`|`# React entry point`|
|`App.tsx`|`# Session restoration, page routing, and report board`|
|`state`||
|`api.ts`|`# Fetch wrapper with silent 401 token refresh and 403`|
|`ban logout`||
|`types.ts`|`# TypeScript interfaces`|
|`index.css`|`# Responsive styling`|
|`components/`||
|`ProtectedRoute.tsx`|`# Route guard blocking guests and non-admins`|
|`Header.tsx`|`# Role-aware top navbar`|
|`FilterBar.tsx`|`# Search, type, and category filter bar`|
|`ItemCard.tsx`|`# Single report card`|
|`ItemList.tsx`|`# Report grid with empty/loading states`|
|`ItemDetailModal.tsx`|`# Report detail popup`|
|`ReportModal.tsx`<br>`pages/`|`# Create / Edit report popup`|
|`LoginPage.tsx`|`# Login form + demo account quick-select buttons`|
|`SignupPage.tsx`|`# Registration form`|
|`ProfilePage.tsx`|`# Name update, password change, and active device`|
|`sessions list`||
|`GroupsPage.tsx`|`# Shared Recovery Teams and member permission`|
|`checkboxes`||
|`AdminPage.tsx`|`# 4-tab Admin Dashboard`|



### **C4. Data Model** 

#### **Figure 3: Database ER Diagram** 

|`erDiagram`|||
|---|---|---|
|`CAMPUS_USER |`|`|--o{`|`CAMPUS_SESSION : "has sessions"`|
|`CAMPUS_USER |`|`|--o{`|`ITEM : "owns reports"`|
|`CAMPUS_USER |`|`|--o{`|`CAMPUS_GROUP : "owns groups"`|
|`CAMPUS_USER |`|`|--o{`|`CAMPUS_GROUP_MEMBER : "joins groups"`|
|`CAMPUS_GROUP`|`||--o{`|`CAMPUS_GROUP_MEMBER : "has members"`|
|`CAMPUS_GROUP`|`||--o{`|`ITEM : "contains reports"`|



```
    CAMPUS_USER ||--o{ CAMPUS_ACTIVITY_LOG : "generates logs"
```

**Table 17:** **`campus_user` Table** 

|**Column**|**Type**|**Key / Constraint**|**Purpose**|
|---|---|---|---|
|`id`|`INTEGER`|`PK`|Unique user ID|
|`name`|`VARCHAR(100)`|`NOT NULL`|Display name|
|`email`|`VARCHAR(150)`|`UNIQUE, INDEX`|Login email|
|`password_hash`|`VARCHAR`|`NOT NULL`|Peppered Argon2<br>password hash|
|`role`|`VARCHAR(20)`|`INDEX`(`'user'`/<br>`'admin'`)|Global RBAC role|
|`is_banned`|`BOOLEAN`|`DEFAULT FALSE`|Immediate suspension<br>flag|
|`ban_reason`/<br>`banned_at`|`VARCHAR`/<br>`TIMESTAMP`|`NULLABLE`|Reason and time of ban|
|`failed_login_co`<br>`unt`/<br>`locked_until`|`INTEGER`/<br>`TIMESTAMP`|`DEFAULT 0`/`NULL`|Brute-force lockout<br>tracking|
|`last_login_at`/<br>`created_at`|`TIMESTAMP`|`UTC`|Login and creation<br>timestamps|



**Table 18:** **`campus_session` Table** 

|**Column**|**Type**|**Key / Constraint**|**Purpose**|
|---|---|---|---|
|`id`|`INTEGER`|`PK`|Session ID|
|`user_id`|`INTEGER`|`FK ->`<br>`campus_user.id,`<br>`INDEX`|<br>Session owner|
|`token_hash`|`VARCHAR(128)`|`UNIQUE, INDEX`|SHA-256 hash of<br>refresh token|
|`ip_address`/<br>`user_agent`|`VARCHAR(64)`/<br>`VARCHAR(255)`|`NOT NULL`|Device IP and browser|
|`created_at`/<br>`last_used_at`/<br>`expires_at`|`TIMESTAMP`|`UTC`|Session lifecycle<br>timestamps|
|`revoked_at`|`TIMESTAMP`|`NULLABLE`|Set when logged out or<br>revoked|



**Table 19:** **`campus_group` &** **`campus_group_member` Tables** 

|**Table**|**Columns**|**Key / Constraint**|**Purpose**|
|---|---|---|---|
|**`campus_group`**|`id`,`name`,|`id PK`,`owner_id`|Shared Recovery Team|



|**Table**|**Columns**|**Key / Constraint**|**Purpose**|
|---|---|---|---|
||`description`,<br>`owner_id`,<br>`owner_name`,<br>`created_at`|`FK ->`<br>`campus_user.id`|metadata|
|**`campus_group_me`**<br>**`mber`**|`id`,`group_id`,<br>`user_id`,<br>`user_name`,<br>`user_email`,<br>`group_role`,<br>`can_create`,<br>`can_edit`,<br>`can_assign`,<br>`joined_at`|`id PK`,`group_id`&<br>`user_id`indexed|Stores<br>`'owner'`/`'member'`<br>and boolean permission<br>flags|



#### **Table 20:** **`item` &** **`campus_activity_log` Tables** 

|**Table**|**Columns**|**Key / Constraint**|**Purpose**|
|---|---|---|---|
|**`item`**|`id`,`type`,`title`,<br>`category`,<br>`location`,<br>`description`,<br>`contact_info`,<br>`status`,`owner_id`,<br>`owner_name`,<br>`group_id`,<br>`assignee_id`,<br>`assignee_name`,<br>`created_at`|`id PK`,`owner_id`&<br>`group_id`indexed|Personal and group<br>lost/found reports|
|**`campus_activity`**<br>**`_log`**|`id`,`user_id`,<br>`user_email`,<br>`action`,<br>`entity_type`,<br>`entity_id`,<br>`detail`,<br>`ip_address`,<br>`user_agent`,<br>`created_at`|`id PK`,`action`&<br>`user_id`indexed|Audit trail of all<br>security and user<br>actions|



### **C5. API Design** 

All endpoints use the `/api` prefix and return consistent JSON errors ( `{"detail": "message"}` ). 

#### **Table 21: REST API Endpoints** 

|**Method & Path**|**Auth**|**Role / Permission **|**Request Body**|**Success / Error**<br>**Codes**|
|---|---|---|---|---|
|`GET`<br>`/api/health`|No|Public|None|`200`|
|`POST`<br>`/api/auth/re`<br>`gister`|No|Public|`UserCreate`|`201`/`409`,`422`|
|`POST`<br>`/api/auth/lo`<br>`gin`|No|Public|`UserLogin`|`200`/`401`,`403`,<br>`429`|
|`POST`<br>`/api/auth/re`<br>`fresh`|Cookie|`refresh_toke`<br>`n`cookie|None|`200`/`401`,`403`|
|`POST`<br>`/api/auth/lo`<br>`gout`|Optional|Active user/cookie|None|`200`|
|`GET`<br>`/api/auth/se`<br>`ssions`|JWT|`CurrentUserD`<br>`ep`|None|`200`/`401`|
|`DELETE`<br>`/api/auth/se`<br>`ssions/{id}`|JWT|`CurrentUserD`<br>`ep`(Own)|None|`200`/`401`,`404`|
|`GET/PUT`<br>`/api/profile`|JWT|`PROFILE_MANA`<br>`GE`|`ProfileUpdat`<br>`e`(on PUT)|`200`/`401`|
|`POST`<br>`/api/profile`<br>`/password`|JWT|`PROFILE_MANA`<br>`GE`|`PasswordChan`<br>`ge`|`200`/`400`,`401`|
|`GET/POST`<br>`/api/items`|JWT|`ITEM_READ_OW`<br>`N`/<br>`ITEM_CREATE`|`ItemCreate`<br>(on POST)|`200`,`201`/`401`,<br>`403`|
|`GET/PUT/`<br>`DELETE`<br>`/api/items/{`<br>`id}`|JWT|Owner, Group<br>Perm, or Admin|`ItemUpdate`<br>(on PUT)|`200`/`401`,`403`,<br>`404`|
|`GET/POST`<br>`/api/groups`|JWT|`GROUP_ACCESS`|`GroupCreate`<br>(on POST)|`200`,`201`/`401`|
|`PATCH/DELETE`<br>`/api/groups/`<br>`{id}`|JWT|Group`owner`or<br>`admin`|`GroupUpdate`|`200`/`403`,`404`|
|`GET/POST`<br>`/api/groups/`<br>`{id}/members`|JWT|Member (GET) /<br>Owner (POST)|`GroupMemberA`<br>`dd`|`200`,`201`/`403`,<br>`404`|
|`PATCH/DELETE`<br>`/api/groups/`<br>`{id}/members`<br>`/{uid}`|JWT|Group`owner`|`GroupMemberP`<br>`ermissionUpd`<br>`ate`|`200`/`400`,`403`,<br>`404`|



|**Method & Path**|**Auth**|**Role / Permission **|**Request Body**|**Success / Error**<br>**Codes**|
|---|---|---|---|---|
|`GET/POST`<br>`/api/groups/`<br>`{id}/items`|JWT|Group Member<br>(`can_create`)|`ItemCreate`|`200`,`201`/`403`|
|`GET`<br>`/api/admin/s`<br>`tats`,`/users`,<br>`/users/{id}`|JWT|`ADMIN_DASHBO`<br>`ARD`|Query filters|`200`/`401`,`403`|
|`PATCH`<br>`/api/admin/u`<br>`sers/{id}/ro`<br>`le`|JWT|`ADMIN_USERS_`<br>`MANAGE`|`{"role":`<br>`"..."}`|`200`/`400`,`403`|
|`POST`<br>`/api/admin/u`<br>`sers/{id}/ba`<br>`n`,`/unban`,<br>`/bulk-ban`,<br>`/bulk-unban`|JWT|`ADMIN_USERS_`<br>`MANAGE`|`{"reason":`<br>`"..."}`|`200`/`400`,`403`|
|`GET`<br>`/api/admin/i`<br>`tems`,`POST`<br>`/api/admin/i`<br>`tems/bulk-`<br>`delete`|JWT|`ADMIN_CONTEN`<br>`T_MODERATE`|`{"ids":`<br>`[...]}`|`200`/`403`|
|`GET`<br>`/api/admin/a`<br>`ctivity`,<br>`/security`|JWT|`ADMIN_AUDIT_`<br>`VIEW`|Query filters|`200`/`403`|
|`GET`<br>`/api/admin/e`<br>`xport/users`,<br>`/items`,<br>`/activity`|JWT|`ADMIN_EXPORT`|Query filters|`200`<br>`(text/csv)`/<br>`403`|



### **C6. JWT Authentication Design** 

#### **Figure 4: JWT Login, Refresh & Logout Sequence** 

```
sequenceDiagram
    participant B as Browser
    participant A as FastAPI (/api)
    participant D as Database
```

```
    B->>A: POST /api/auth/login (email, password)
    A->>D: Verify Pepper+Argon2 hash & save SHA-256 refresh session
    A-->>B: 200 OK (15m JWT) + Set-Cookie: refresh_token (HttpOnly, 7d)
```

```
    B->>A: GET /api/items (Authorization: Bearer <JWT>)
    A-->>B: 200 OK (Own reports)
```

```
    Note over B,A: After 15 minutes (JWT expired)
    B->>A: GET /api/items (Expired JWT)
    A-->>B: 401 Unauthorized
    B->>A: POST /api/auth/refresh (HttpOnly refresh_token cookie)
    A->>D: Verify & rotate token_hash in campus_session
    A-->>B: 200 OK (New 15m JWT + rotated cookie)
    B->>A: Retry GET /api/items -> 200 OK
```

```
    B->>A: POST /api/auth/logout
    A->>D: Set campus_session.revoked_at = now
    A-->>B: 200 OK + Clear cookie
```

**Table 22: JWT Access Token Claims** 

|**Claim**|**Value**|**Purpose**|
|---|---|---|
|`sub`|`"3"`|User ID primary key|
|`email`|`"student@campus.edu"`|User email|
|`role`|`"user"`or`"admin"`|Global role|
|`iat`/`exp`|UNIX timestamps|Issued-at and 15-minute expiry<br>(`iat + 900`)|



- **Lifetimes & Storage:** The access token lives for **15 minutes** (signed with HS256 using `JWT_SECRET` ) so a stolen token expires quickly. The refresh token lives for **7 days** inside an `HttpOnly` , `SameSite=Lax` cookie so JavaScript (XSS) cannot read it, while the `Authorization: Bearer` header + CORS protects against CSRF. 

- **Server & Frontend Checks:** On every request, `security.py` checks the JWT signature, expiry, that the user exists in `campus_user` , and that `user.is_banned == False` . On a `401` , `frontend/src/api.ts` calls `/api/auth/refresh` once and retries; if refresh fails, it logs the user out cleanly. 

### **C7. RBAC Design** 

#### **Figure 5: Request Authorization Flow** 

##### `flowchart TD` 

```
    Req["Incoming Request"] --> JWT["1. security.py: Verify JWT & Check
user.is_banned"]
```

```
    JWT -->|Invalid / Expired| E401["401 Unauthorized + Log 'token_rejected'"]
    JWT -->|is_banned == True| E403B["403 Account Banned + Log 'rbac_denied'"]
    JWT -->|Valid User| Perm["2. rbac.py: require_permission(Permission.*)"]
    Perm -->|Role Lacks Perm| E403["403 Forbidden + Log 'rbac_denied'"]
    Perm -->|Permitted| Scope{"3. Resource Type?"}
    Scope -->|Personal Item| Own["enforce_item_ownership (owner_id == user.id)"]
    Scope -->|Group Item / Group| Grp["enforce_group_permission (owner or
can_create/edit/assign)"]
    Scope -->|Profile / Admin| Run["4. Run Handler"]
    Own -->|Not Owner| E403
    Grp -->|Missing Flag| E403
    Own -->|Owner / Admin| Run
    Grp -->|Allowed| Run
```

####  **Reusable Permission Check (** **`backend/rbac.py` ):** 

```
def has_permission(user: User, permission: str) -> bool:
    allowed = ROLE_PERMISSIONS.get(user.role, USER_PERMISSIONS)
    return permission in allowed
def require_permission(permission: str) -> Callable[..., User]:
    def _dependency(
        request: Request, session: SessionDep, current_user: CurrentUserDep,
    ) -> User:
        if not has_permission(current_user, permission):
            record_security_event(
                session, request, "rbac_denied",
                f"Denied permission '{permission}' on {request.method}
{request.url.path}",
                user_id=current_user.id, user_email=current_user.email,
            )
            raise HTTPException(status_code=403, detail=f"Insufficient
permissions: requires '{permission}'.")
        return current_user
    return _dependency
```

- **Query-Level Ownership & Extensibility:** `list_user_scoped_reports` filters SQL directly with `where(Item.owner_id == current_user.id)` , and `enforce_item_ownership` blocks reading, editing, or deleting another user's item ( `403` ). Adding a new permission only requires adding a constant to `Permission` and `ROLE_PERMISSIONS` in `backend/rbac.py` . 

### **C8. Admin Dashboard Design** 

**Figure 6: Admin Pages and Endpoints** 

```
flowchart TD
    Admin["AdminPage.tsx"] --> T1["Tab 1: Users & Roles"]
    Admin --> T2["Tab 2: Content Moderation"]
    Admin --> T3["Tab 3: Activity Audit Log"]
    Admin --> T4["Tab 4: Security Monitor"]
    T1 --> E1["/api/admin/stats, /users, /users/{id}, /role, /ban, /bulk-ban,
/export/users"]
    T2 --> E2["/api/admin/items, /items/bulk-delete, /groups, /export/items"]
    T3 --> E3["/api/admin/activity, /export/activity"]
    T4 --> E4["/api/admin/security"]
```

**Table 23: Admin Tabs, Endpoints, Filters, Actions & Audit Events** 

|**Tab**|**Endpoints**|**Filters &**<br>**Pagination**|**Actions**|**Audit Events**|
|---|---|---|---|---|
|**Users & Roles**|`/api/admin/`<br>`stats`,`/users`,<br>`/users/{id}`,<br>`/role`,`/ban`,<br>`/unban`,<br>`/bulk-ban`,<br>`/export/user`<br>`s`|Search<br>(`name`/`email`),<br>role filter, status<br>filter, pagination<br>(`limit=10`)|Inspect user<br>drawer, change<br>role, ban/unban<br>with reason,<br>revoke sessions,<br>bulk ban/unban,<br>CSV export|`role_change`,<br>`user_ban`,<br>`user_unban`,<br>`session_revo`<br>`ke`,<br>`data_export`|
|**Content**<br>**Moderation**|`/api/admin/`<br>`items`,|Search, type<br>(`lost`/`found`),|Delete single or<br>bulk reports,|`item_delete`,<br>`group_delete`,|



|**Tab**|**Endpoints**|**Filters &**<br>**Pagination**|**Actions**|**Audit Events**|
|---|---|---|---|---|
||`/items/bulk-`<br>`delete`,<br>`/groups`,<br>`/export/item`<br>`s`|category|delete group,<br>export items CSV|`data_export`|
|**Activity Audit**<br>**Log**|`/api/admin/`<br>`activity`,<br>`/export/acti`<br>`vity`|Search (`email`,<br>`IP`,`detail`),<br>event type filter|Filter audit events,<br>export filtered<br>activity CSV|`data_export`|
|**Security Monitor**|<sup>`/api/admin/`</sup><br>`security`|Grouped by IP<br>and by account|View failed login<br>counts and recent<br>token/RBAC<br>rejections|Read-only view|



### **C9. Security Design and Threats** 

#### **Table 24: Threats and Defences** 

|**Threat**|**Defence in Our Code**|**File**|
|---|---|---|
|**Password DB Leak**|Passwords mixed with server<br>`PASSWORD_PEPPER`(HMAC-<br>SHA256) before Argon2<br>hashing|`backend/security.py`|
|**Stolen Access Token**|Short 15-minute expiry + DB<br>ban check on every request|`backend/security.py`|
|**Stolen Refresh Token**|`HttpOnly`cookie, SHA-256<br>hashed in DB, rotated on<br>refresh, revocable anytime|`backend/routers/`<br>`auth.py`,`service.py`|
|**Brute-Force Login**|5 wrong attempts lock the<br>account for 15 minutes (`429`<br>`Too Many Requests`)|`backend/service.py`|
|**Privilege Escalation**|Signup hardcodes<br>`role="user"`; role change<br>requires<br>`ADMIN_USERS_MANAGE`|`backend/models.py`,<br>`rbac.py`|
|**Reading Others' Data**|SQL query filters by<br>`owner_id ==`<br>`current_user.id`+<br>`enforce_item_ownershi`<br>`p`(`403`)|`backend/service.py`,<br>`rbac.py`|
|**XSS**|React JSX auto-escaping,<br>`HttpOnly`refresh cookies,`X-`<br>`Content-Type-Options:`<br>`nosniff`|`backend/main.py`,<br>`frontend/`|



|**Threat**|**Defence in Our Code**|**File**|
|---|---|---|
|**CSRF & CORS Misuse**|Protected calls require<br>`Authorization: Bearer`<br>header; CORS restricted to<br>`FRONTEND_ORIGIN`|<br>`backend/main.py`,<br>`api.ts`|
|**Secrets in Git**|`.gitignore`excludes`.env`<br>and`*.db`; only<br>`backend/.env.example`is<br>committed|<br>`.gitignore`,<br>`backend/.env.example`|
|**SQL Injection**|Parameterized SQLModel<br>`select().where()`queries<br>everywhere|`backend/crud.py`|
|**Over-Posting Fields**|Input schemas (`UserCreate`,<br>`ItemCreate`,`ItemUpdate`)<br>exclude`role`,`is_banned`,<br>`owner_id`|`backend/models.py`|



### **C10. Deployment on Vercel** 

#### **Figure 7: Deployment Layout** 

```
flowchart LR
    User["Browser"] --> Vercel["Vercel Routing (vercel.json)"]
    Vercel -->|"/* (Static SPA)"| Vite["React Frontend Build"]
    Vercel -->|"/api/* (Serverless)"| FastAPI["FastAPI Backend
(backend/main.py)"]
    FastAPI -->|"DATABASE_URL (NullPool)"| DB[("Hosted PostgreSQL")]
```

#### **Table 25: Environment Variables on Vercel** 

|**Variable Name**|**Purpose**|
|---|---|
|`DATABASE_URL`|Hosted PostgreSQL connection string|
|`JWT_SECRET`|Secret key used to sign and verify HS256 access<br>tokens|
|`PASSWORD_PEPPER`|Server-side secret mixed with passwords before<br>Argon2 hashing|
|`ACCESS_TOKEN_MINUTES`|Access token lifetime in minutes (`15`)|
|`REFRESH_TOKEN_DAYS`|Refresh session lifetime in days (`7`)|
|`FRONTEND_ORIGIN`|Allowed production frontend URL for CORS|



### **C11. Testing** 

We run automated API tests with `pytest` and FastAPI `TestClient` in `backend/tests/test_auth_and_rbac.py` , plus manual verification on the live Vercel deployment. 

#### **Table 26: Automated Test Results** 

|**Test Name**|**Requirement(s)**|**What It Checks**|**Result**|
|---|---|---|---|
|`test_login_and_`<br>`refresh_flow`|`FR-AUTH-03`,`05`|Login returns 15m<br>JWT,<br>`/api/auth/refre`<br>`sh`rotates cookie,<br>`/logout`revokes<br>session (`401`after)|**PASSED**|
|`test_token_reje`<br>`ction_missing_t`<br>`ampered_and_exp`<br>`ired`|`FR-AUTH-04`,<br>`AC-01`,`02`|Missing header, bad<br>header, tampered<br>signature, and expired<br>JWT all return`401`|**PASSED**|
|`test_brute_forc`<br>`e_login_lockout`|`FR-AUTH-07`,`AC-07`|5 wrong password tries<br>lock the account with<br>`429 Too Many`<br>`Requests`|**PASSED**|
|`test_rbac_rule_`<br>`1_normal_user_d`<br>`enied_admin_end`<br>`points`|`FR-RBAC-01`,`FR-`<br>`ADM-*`|Normal user gets`403`<br>on<br>`/api/admin/stat`<br>`s`,`/users`, and<br>`/export/users`|**PASSED**|
|`test_rbac_rule_`<br>`2_ownership_pre`<br>`vents_modifying`<br>`_other_users_re`<br>`cord`|`FR-RBAC-02`,`AC-03`|User sees only own<br>items and gets`403`on<br>`GET`,`PUT`, or`DELETE`<br>of another user's item|<br>**PASSED**|
|`test_rbac_rule_`<br>`3_fine_grained_`<br>`group_permissio`<br>`ns`|`FR-RBAC-03–05`,<br>`AC-04`|Sajid (`can_create`<br>only) creates (`201`) but<br>fails edit (`403`);<br>Ammar (`can_edit`)<br>edits (`200`) but fails<br>owner delete/rename<br>(`403`); Owner deletes<br>(`200`)|**PASSED**|
|`test_rbac_rule_`<br>`4_admin_safety_`<br>`and_immediate_b`<br>`an_enforcement`|`FR-RBAC-06–07`,<br>`AC-05`,`06`|Admin self-demote<br>(`400`) and banning<br>another admin (`400`)<br>fail; banned user gets<br>`403`on very next<br>request|**PASSED**|



## **Part D. Evidence: Screenshots of the Running Application** 

### **D1. User Side** 

- **[S1: Signup and login page with validation error]** — Proves form validation and auth landing guard ( `FR-AUTH-01` , `FR-AUTH-03` ). 



<!-- Start of picture text -->
@ Campus Lost & Found op 01 comected<br>i)<br>Sign In to Campus Lost & Found<br><!-- End of picture text -->

- **[S2: Logged-in user's main board showing only their own reports]** — Proves personal report ownership scoping ( `FR-DOMAIN-02` , `FR-RBAC-02` ). 



<!-- Start of picture text -->
G23 campus-lost-found-management-sy.. Q@o + 4) School:<br>e Campus Lost & Found<br>Secure Campus Belongings Recovery & Team Desks<br>| El My Reports © Recovery Teams 2 Ammar Rahman USER Log Out<br>API Connected + Report Lost + Report Found<br>All Reports (1) @ Lost Items @ Found Items<br>Search by title or description All Categories ~ UO Refresh<br>Lost ACTIVE<br>Blue Leather Notebook & Pen<br>Auditorium Row C+ # Books & Stationen<br>Contains handwritten lecture notes for Web Application course.<br>& Ammar Rahman + 11/23/2026 View Edit Delete<br><!-- End of picture text -->

- **[S3: Profile page password change rejecting wrong current password]** — Proves current password verification ( `FR-AUTH-08` ). 



<!-- Start of picture text -->
e Campus Lost & Found<br>Secure Campus Belongings Recovery & Team Desks<br>El My Reports @ Recovery Teams | & AmmarRahman user | Log Out<br>2 Account Profile & Change Password<br>Signed in as ammar@campus.edu Current password required; failed attempts are<br>audited<br>mole useR [cincsiis/25/2028) Current password is incorrect.<br>Display Name Current Password<br>Ammar Rahman veceeee<br>Update Password<br><!-- End of picture text -->

- **[S4: Active device sessions list on Profile page with Revoke button]** — Proves session tracking and revocation ( `FR-AUTH-06` ). 



<!-- Start of picture text -->
© Active Login Sessions (2)<br>Manage devices logged into your account via HttpOnly refresh tokens<br>Revoke Other Devices Log Out Everywhere<br>Session IP Address Browser / Device Last Activity Action<br>#6(Current Mozilla/5.0i w (Windows NT 10.0; .Win64; x64)J 11/24/2026,<br>Chrome/159.0.0.0 Safari/537.36 —<br>Device) 103.72.212.68 — AppleWebkit/537.36 (KHTML, like Gecko) 10:30:10 PM Revoke<br>#2 103.72.212.6872.212. — Mozilla/5.07 . 9:50:0811/23/2026,PM Revoke<br>a, eee rene TY ——.<br><!-- End of picture text -->

- **[S5: Recovery Teams page showing group members and mixed permissions]** — ( `FRRBAC-03` , `FR-RBAC-04` ). 



<!-- Start of picture text -->
© My Recovery Teams (1) it Main Library Lost & Found Team<br>aboratewith department desks and with fine-grained Sakiftshmam+ Your Permission + (Create, Edit, Dele<br>Main Library Lost & Found Team owner Delete Team<br>Shared recovery desk for library volunteers and student assistants. +3 member(s)<br>Main Library Lost & Found Team Rename<br>* Greate New Recovery Team 42 Team Members & Granular Permissions (3)<br>Team / Desk Name *<br>5, CSE Lost & Found Desk Member Role Create Edit Assign Action<br>student@campusedu<br>esetption Sakif Ishmam owner<br>nresponst Jesk hour ‘Arnmae Rahman . seemove<br>ammar@campusedu  “EMBE<br>Create Team (You become Owner) saidsaid@compuseduAhmed MEMBER c Remove<br>+ Add Member by Campus Email (Owner Only)<br>9, ammar@campus edu ‘Add Member<br>can Create OCan Edit OCan Assign<br><!-- End of picture text -->

- **[S6: Action denied by RBAC with 403 in browser Network tab]** — Proves server-side `403 Forbidden` enforcement ( `FR-RBAC-01` , `FR-RBAC-04` ). 



<!-- Start of picture text -->
ape nge ent ee<br><!-- End of picture text -->

### **D2. Security Behaviour** 

- **[S7: DevTools Network tab showing masked** **`Authorization: Bearer` header and expired token returning** **`401` ]** — Proves JWT verification ( `FR-AUTH-04` ). 



<!-- Start of picture text -->
py Campus Lost& Found @ | % A | Gkeepbes | G Ousblecache| Notwottiog ~ ||<br>My Reports | © Recovery Teams SwidANmed USER Log Out v ik eat ‘<br>ost nerve 1 profi owe Sete<br>onda Motorcycle Key with Red Keychain: hevraanbtaras leavinasine .<br>BO =| we ©) ¥ Fite ‘ + | AB a<br><!-- End of picture text -->

- **[S8:** **`POST /api/auth/refresh` returning new access token and** **`HttpOnly` cookie in Application tab]** — Proves silent token rotation ( `FR-AUTH-05` ). 



<!-- Start of picture text -->
Campus Lost & Found easesO|% & | raatBieoplon See aeOh ot Be sit@O i 1€x<br>da MotorcycleKey with Red Keychain See<br><!-- End of picture text -->



<!-- Start of picture text -->
Campus Lost & Found @ira ke No throttling Sit<br>‘propor | © Recovery Toums —% SaldAnmed SEH Log Out v ik \ Mor fiers ><br>nda Motorcycle Key with Red Keychain cmerenn Troe [emol —<br><!-- End of picture text -->

- **[S9: Banned user rejected on their very next request with** **`403 Account banned` ]** — Proves immediate ban check ( `FR-RBAC-07` ). 



<!-- Start of picture text -->
G 2% _campus-lost-found-management-system.vercel.app<br>Sign In to Campus Lost & Found<br>Access your profile, manage your reports, collaborate in recovery<br>teams, or administer the system<br>Account banned: spam test<br>Campus Email Address<br>sajid@campus.edu<br>Password<br>+ Select Instructor Demo Account (Prefills Email):<br>© Chief Admin — admin@campus.edu<br>‘© Desk Moderator — moderator@campus.edu<br>® Sakif Ishmam (Group Owner) —<br>student@campus.edu<br>‘®@ Ammar Rahman (Create/Edit/Assign) —<br>ammar@campus.edu<br>‘@ Sajidoa AhmedTianries(Create eteOnly) —<br><!-- End of picture text -->

### **D3. Admin Dashboard** 

- **[S10: Admin overview statistics with 7-day counts]** — Proves `GET` 

   - `/api/admin/stats` ( `FR-ADM-01` ). 



<!-- Start of picture text -->
e Campusa Lost &:  Found ;<br>© Campus Administration & Security Center w Refresh Dashboard<br>5 2 ° 6 3 3 1<br><!-- End of picture text -->

- **[S11: Admin users list with search, role/status filters, and pagination]** — Proves `GET /api/admin/users` ( `FR-ADM-02` ). 



<!-- Start of picture text -->
$2) campus-lost-found-management-system.vercelapp Qr |<br>82 Registered Accounts (5) J Export Users CSV<br>s me or om Al Roles Y AlStatuses. v<br>Select 1D Name<br>& Email Role Status Actions<br>* 1 Campusadmin@campuseduChief Admin (os) sown acre1 Inspect°<br>#2 aaa mene ADMIN active Inspect Make User<br>3 Sakiftshmam user active Inspect Make Admin Ban<br>student@campus.edu<br>4 Aa m mar@cmar R a mpuseduhman user active Inspect Make Admin Ban<br>as Ss a lidjid@caAn m pus.edued user active Inspect Make Admin Ban<br>Showing1-5 of 5<br><!-- End of picture text -->

- **[S12: User detail inspector modal showing reports, groups, sessions, and recent activity]** — Proves `GET /api/admin/users/{id}` ( `FR-ADM-02` ). 



<!-- Start of picture text -->
2) campus-lost-found-management-system.vercelapp Qr<br>Showing1-5 085<br>& User Detail: Ammar Rahman (ammar@campus.edu) *<br>Role: USER Reports:1 Groups: Active Sessions: 1<br>Revoke All 1 Active Session(s)<br>Submitted Reports (1)<br>‘#3 [LOST] Blue Leather Notebook & Pen — Auditorium Row C<br>Group Memberships (1)<br>‘+ Main Library Lost & Found Team (member)<br>Recent Activity (14)<br>* logout: User logged out and invalidated session (11/24/2026, 10:35:51 PM)<br>* session revoke: Revoked session #6 (11/24/2026, 10:34:43 PM)<br>* password change failed: Failed password change: current password incorrect (11/24/2026,<br>10:31:46 PM)<br>* login: User signed in successfully (11/24/2026, 10:30:10 PM)<br>© failed login: invalid password or unknown email for ‘ammar@campus.edu' (11/24/2026,<br>10:25:50 PM)<br>* logout: User logged out and invalidated session (11/24/2026, 10:25:44 PM)<br>* logout: User logged out and invalidated session (11/24/2026, 10:25:44 PM)<br>© login: User signed in successfully (11/24/2026, 10:18:03 PM)<br>* failed login: Invalid passwordor unknown email for ‘ammar@campus.edu' (11/24/2026,<br>10:17:55 PM)<br>* item delete: Deleted report #9 (t2') (11/23/2026, 9:50:10 PM)<br><!-- End of picture text -->

- **[S13: Ban dialog with reason and resulting** **`BANNED` status badge]** — Proves `POST /api/admin/users/{id}/ban` ( `FR-ADM-03` ). 



<!-- Start of picture text -->
ea e 23 campus-lost-found-management-system.vercelapp Qx & incd<br>campus-lost-found-management-system.vercel.app says<br><!-- End of picture text -->



<!-- Start of picture text -->
< © —&_ campus-lost-found-management-system.vercel.app Qr & incognito<br>PET © content moderation Activity Audit Log Security Monitor<br>£2 Registered Accounts (5) 1 Export Users CSV<br>Search name or ema All Roles ~  allStatuses. ¥<br>Select. 1D Name<br>& Email Role status Actions<br>“1 Campus Chief Admin (vou) amin active Inspect<br>edmin@campus.edu<br>* Security Desk Moderator amin Inspect Make User<br>2 moderator@campus.edu aca =<br>ee eee<br>#3<br>student@campus.edu USER ACTIVE Inspect Make Admin Ban<br>+4 Ammarammar@campus.eduRahman user active Inspect Make Admin Ban<br>+s Sajid‘sajid@campus.edu Ahmed user BANNED m(spam tert J Inspect Make Admin Unban<br><!-- End of picture text -->

- **[S14: Content Moderation list with filters and bulk delete]** — Proves report and group 

moderation ( `FR-ADM-04` ). 



<!-- Start of picture text -->
© All Reports Moderation (6) SASSY GUSSET<br>© Recovery Teams Moderation (1)<br><!-- End of picture text -->

- **[S15: Activity Audit Log with filters showing user, event, IP address, and browser]** — Proves `GET /api/admin/activity` ( `FR-ADM-05` ). 



<!-- Start of picture text -->
System Activity & Audit Log (39) 1 Export Activity CSV<br>Search user oma, Pad AALEvents ¥<br>Time User Event Detail 1P & Browser<br>pM<br>AppleWedkit/S<br>103.72.212.73<br>19/24/2026, 11:08:55<br>wv seid@campusedy fale login Banned user attempted login: spam test wesre<br>w03.72.212.73<br>ssyparnoee 0042<br><!-- End of picture text -->

- **[S16: Security Monitor showing failed logins by IP and account plus token rejections]** 

   - Proves `GET /api/admin/security` ( `FR-ADM-06` ). 



<!-- Start of picture text -->
campus-lost-found-management-system.vercel.app Q<br>43 Users & Roles @ Content Moderation Activity Audit Log [aReetNee<br>& Security Monitoring (Failed Logins & Denied Requests)<br>Failed Logins by IP Address Failed Logins by Account<br>IP Address Attempts Account Email Attempts<br>103.72.212.72 5 ammar@campus.edu 7<br>103.72.212.73 1 sajid@campus.edu 1<br>103:72.212.68 1<br>103:72.212.78 1<br>Recent Security Events (Rejected Tokens, RBAC Denials & Failed Logins)<br>Time Account Event Detail P<br>11/25/2026, 12:37:59 PM ammar@campus.edu failed login Account locked for 15m after5 failed logins 103.72.212.72<br>11/25/2026, 12:37:57 PM ammar@campus.edu failed login Invalid password or unknown email for ‘ammar@campus.edu' 103.72.212.72<br>11/25/2026, 12:37:54 PM ammar@campus.edu failed login Invalid password or unknown email for ‘ammar@campus.edu' 103.72.212.72<br>11/25/2026, 12:37:52PM  ammar@campus.edu failed login Invalid password or unknown email for ‘ammar@campus.edu' 103.72.212.72<br>11/25/2026, 12:37:43 PM ammar@campus.edu failed_login Invalid password or unknown email for 'ammar@campus.edu' 103.72.212.72<br><!-- End of picture text -->

- **[S17: Downloaded CSV export opened]** — Proves filter-aware CSV export ( `FRADM-06` ). 



<!-- Start of picture text -->
5% Autosave of)@ Deo users export V<br>File Home Insert Draw Page layout Formulas Data Review View Automate Help<br>Pastea] fa‘ ~ AptosBIU-H-+ Narrow v|11_ @-4 ¥|AAyAY S====== = [J] ~ | Be2 General$ ~ 9% 9/80.092) | Conditional Formatasewetas Cellcel ="EOF<br>- ¥ Formatting’ Tablev Styles- [BiFo|<br>Clipboard Font 5 Alignment 5 Number 5 Styles q<br>as Yi fe<br>2 id A name 8 email c role D is bannedE ban_reasoF created_atGc E 1 x i<br>1. Campus<br>2 SecurityChief Admin admin@campus.edu admin FALSE 2026-11-23115:46:33.260347+00:00<br>45 34 AmmarSakiflshmam RahmanDesk Moderator moderastudenammar@campus.edu t @campus.eduor@campus.edu adminuseruser FALSE FALSE 2026-11-23 2026-11-23715:46:33.956516+00:0071 15:4 6:33.528396+00:006:33.749597+00:00<br>é 5 Sajid Ahmed sajid@campus.edu user FALSE 2026-11-23115:46:34.160115+00:00<br>1<br>0<br>B<br>3<br>4<br>15<br><!-- End of picture text -->

### **D4. Deployment** 

- **[S18: Vercel dashboard showing production deployment and live domain]** — Proves live deployment from `main` ( `NFR-05` ). 



<!-- Start of picture text -->
© 4% vercelcom/2220681-1731/campus-lost-found-management-system Qe we Bschoor :<br>po-rrare. Ht campus-iotfound-mana Overview + createNew Agent<br>campus-lot-ound-management-system-1b6tan5e-2220681-1731verceL 360<br>— © Ready 23 ag0by shmam2z22@<br>Recommendations. 26 beervabity Anatytics<br><!-- End of picture text -->

- **[S19: Vercel environment variable names with values hidden]** — Proves secrets stored 

- in environment variables ( `NFR-01` ). 



<!-- Start of picture text -->
* Environment Variables Link Shared Variable<br>DATARASE-unL_unPooLED e<br>2A ‘<br>rable POSTORES_PRISHA.URL e<br>POSTORES_URL_NON. POOLING e<br><!-- End of picture text -->

- **[S20: Live FastAPI** **`/api/docs` Swagger page on Vercel]** — Proves live API documentation ( `Table 1` ). 



<!-- Start of picture text -->
© &%_campus-lost-found-management-system.vercel.app/api/docs Q we — B schoo<br>Japh/Stens/ {Sten 44) UF oto — nk ; jn v<br>auth A<br><!-- End of picture text -->

### **D5. Tests** 

- **[S21: Terminal output showing all 7 automated tests passing]** — Proves automated test verification ( `Table 26` ). 



<!-- Start of picture text -->
(OM Administrator: Windows X + ~<br>Copyright (C) Microsoft Corporation. All rights reserved<br>PS C:\Users\User> cd E:\Git\Campus-Lost-Found-Nanagement-System-main\backend; .\.venv\Scripts\python.exe -m pytest tests<br>/test_auth_and_rbac.py<br>sessnnssasScessssssssssssssssssssssscssssssscsss= test session starts sssssssssssssssscsssssesssesssssscsssssessssssss=<br>platform win32 -- Python 3.12.8, pytest-9.2.0, pluggy-1.7.0 ~~ E:\Git\Campus-Lost-Found-Management-System-main\backend\<br>venv\Scripts\python exe<br>cachedir: .pytest_cache<br>rootdir: E:\Git\Campus-Lost-Found-Management-System-mai<br>configfile: pyproject.toml<br>plugins: anyio-4.15.1<br>collected 7 items<br>tests\test_auth_and_rbac.tests\test_auth_and_rbac.tests\test_auth_and_rbac. py:py:py: :test_login_and_refresh_flow:test_token_rejection_missing_tampered_and_expired:test_brute_force_login_lockout<br>tests\test_auth_and_rbac. py: :test_rbac_rule_1_normal_user_denied_admin_endpoints<br>tests\test_auth_and_rbac.py: :test_rbac_rule_2_ownership_prevents_modifying_other_users_record<br>| tests\test_auth_and_rbac. py: :test_rbac_rule_3_fine_grained_group_permissions<br>tests\test_auth_and_rbac. py: :test_rbac_rule_4_admin_safety_and_immediate_ban_enforcemen<br>venv\Lib\site-packages\fastapi\testclient.py:1E:\Git\Campus-Lost-Found-Management-System-main\backend\ .venv\Lib\site-packages\fastapi\testclient.py:1: StarletteDep:<br>ecationliarning: Using ‘httpx’ with ‘starlette.testclient’ is deprecated; install ‘httpx2' instead<br>from starlette.testclient import TestClient as TestClient # noga<br>-- Docs: https://docs.pytest org/en/stable/hon-to/capture-warnings html<br>, 1 warning<br>| PS E:\Git\Campus-Lost-Found-Managenent-System-main\backend><br><!-- End of picture text -->

## **Part E. Project Management and Conclusion** 

### **E1. How to Run and Test** 

1. **Prerequisites:** Python 3.10+, Node.js 18+, and Git. 

2. **Clone Repository:** 

```
git clone https://github.com/Ishmam2222/Campus-Lost-Found-Management-System.git
cd Campus-Lost-Found-Management-System
```

3. **Environment File:** Copy `backend/.env.example` to `backend/.env` : 

```
DATABASE_URL=sqlite:///./lost_and_found.db
JWT_SECRET=local-dev-jwt-secret-key
PASSWORD_PEPPER=local-dev-password-pepper
ACCESS_TOKEN_MINUTES=15
REFRESH_TOKEN_DAYS=7
FRONTEND_ORIGIN=http://localhost:5173
```

#### 4. **Run Backend & Tests:** 

```
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
pytest -v
fastapi dev main.py
```

#### 5. **Run Frontend:** 

```
cd ../frontend
npm install
npm run dev
```

6. **Demo Accounts:** The backend seeds test accounts for **`admin`** and **`user`** roles plus the 

shared group _Main Library Lost & Found Team_ . Per course rules, the user IDs and passwords are listed only in the **PDF Report Appendix** and the submission form, never in this repository file. 

### **E2. GitHub Work** 

All commits and pull requests were completed under my GitHub account ( `Ishmam2222` ). 

**Table 27: GitHub Issues and Pull Requests** 

|**Issue**|**Pull Request / Branch**|**What It Delivered**|
|---|---|---|
|**Issue #1 / #2**|**PR #1, #3**(`docs-project-`<br>`specifications`)|Initial project scaffold, Vercel<br>configuration, and mid-term<br>PRD/SRS/TDD docs|
|**Issue #5**|**PR #6, #7**(`CRUD/5-`<br>`report-crud`)|Lost & Found CRUD API<br>(`/api/items`), database<br>models, and React board UI|
|**Issue #8**|**PR #8**(`feature/8-auth-`<br>`and-rbac`)|JWT auth, Pepper+Argon2<br>hashing, refresh cookie rotation,<br>`rbac.py`, ownership rules,<br>shared Recovery Teams, 4-tab<br>Admin Dashboard, audit log,<br>security view, CSV exports,<br>and 7 automated tests|



### **E3. Limitations** 

- **No Image Uploads:** Reports use text descriptions instead of photo uploads to avoid external cloud storage dependencies. 

- **No Email Password Reset:** Password changes require the current password while logged in because no external SMTP server is connected. 

- **Manual Audit Refresh:** The Admin Audit Log updates on tab load or click rather than live WebSockets on serverless functions. 

### **E4. Future Work** 

1. **Photo Uploads with Cloud Storage:** Allow users to attach item photos using Supabase Storage. 

2. **Automatic Lost/Found Match Alerts:** Notify a user when a newly found item matches their lost item's category and location. 

3. **QR Code Tags for Desk Items:** Generate printable QR labels for physical items stored at campus recovery desks. 

### **E5. Sources, Libraries and Tools** 

**Table 28: Sources, Libraries and Tools** 

|**Name**|**Version / Link**|**What It Was Used For**|
|---|---|---|
|**FastAPI & SQLModel**|`>=0.141`/`>=0.0.22`<br>(https://fastapi.tiangolo.com<br>)|Backend REST API, ORM<br>models, validation, and<br>`/api/docs`|
|**PyJWT & pwdlib[argon2]**|`>=2.8`/`>=0.2`<br>(https://pyjwt.readthedocs.io<br>)|HS256 JWT tokens and Argon2<br>password hashing|
|**React, TypeScript & Vite**|`18.3`/`5.6`/`6.x`<br>(https://react.dev<br>)|Frontend SPA components,<br>route guards, and bundling|
|**Pytest & HTTPX**|`>=8.0` (https://docs.pytest.org<br>)|Automated API testing for auth<br>and RBAC|
|**CSE309 Reference**<br>**(****`smarter-todo`)**|https://github.com/sayef-<br>reyadh-labs/smarter-todo|Reference for course JWT,<br>RBAC, and report format<br>expectations|
|**AI Assistant (Gemini /**<br>**Antigravity)**|Coding & documentation<br>assistant|Helped structure FastAPI<br>routers, React admin tabs, and<br>markdown tables; all RBAC<br>rules, demo accounts, and tests<br>were reviewed and verified by<br>me|



## **Appendix. Test Accounts and Video Demo** 

- **Video Demo Link (5–8 minutes on live Vercel URL):** _(Provided in the Final Submission Form and PDF Report Appendix)_ 

- **Test Accounts & Passwords:** Per course rules ( _never commit passwords to GitHub_ ), the full table of **2 Admin accounts** , **3 Normal User accounts** , their **passwords** , and the **Main Library Lost & Found Team** member permissions is included in the **Appendix of the submitted PDF Report** and the **Final Submission Google Form** . 

