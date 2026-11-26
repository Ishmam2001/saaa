# 9. Code Review Tools and Findings

To enforce SRS Requirements **`MAIN-03`** (linter-checked coding standard), **`MAIN-04`** (pull-request peer review and passing automated CI), and **`SEC-07`** (security verification), the ClinicQ team combined **three automated static analysis tools** with **GitHub Pull Request reviews and GitHub Actions CI**.

---

## 9.1 Static Analysis & Code Review Tools Used

| Tool | Purpose in ClinicQ | Command Executed | Final Codebase Result |
| :--- | :--- | :--- | :--- |
| **1. Ruff** (`pyproject.toml` + GitHub Actions CI) | Enforces PEP 8 (`E`, `W`), Pyflakes bug checks (`F`), import sorting (`I`), and Google docstrings (`D`) on every push and PR. | `ruff check .` | **`All checks passed!`** (0 errors across all 7 apps & `docs/`) |
| **2. Pylint + `pylint-django`** | Deep AST static analysis for unused arguments, misplaced imports, method-override mismatches, and exception handling. | `pylint --load-plugins=pylint_django --ignore=migrations accounts booking consultation notifications queueing reports scheduling` | **Rated `9.97 / 10`** |
| **3. Bandit** (PyCQA Security Scanner) | Scans Python AST for security vulnerabilities (`CWE-259` hardcoded secrets, SQL injection, weak `random` PRNG, unsafe `eval`/subprocess). | `bandit -r accounts booking consultation notifications queueing reports scheduling config -x "*/tests*,*/test_*,*/migrations/*"` | **`0` issues in `4,783` lines of production code** (`0 High`, `0 Medium`, `0 Low`) |
| **4. GitHub Pull Requests & Actions CI** | Enforces PR checklist (`.github/pull_request_template.md`), peer review, and automated `ruff check .` + `python manage.py test` runs before/on merge. | `.github/workflows/ci.yml` across 33 PRs | **CI Passing** (Run `#45427123068` & PR `#33` Run `#8`) |

### Static Analysis Terminal Output (`ruff check .`, `bandit -r .`, `pylint`)

![Real_Terminal_Ruff_Bandit_Pylint](https://github.com/user-attachments/assets/b1c47fbf-fb2b-4194-96bf-27ffdcf9da7d)

---

## 9.2 Six Real Findings, Root-Cause Explanations, and Corrective Actions

Below are **six genuine issues** detected by Ruff, GitHub Actions CI, Pylint, Bandit, and PR reviews on the ClinicQ repository, each paired with its real screenshot, root cause, and exact commit/PR fix:

---

### Finding 1 — Unsorted Imports (`I001`) and Missing Trailing Newline (`W292`) Caught on PR #33
- **Tool:** **Ruff (`ruff check .`) in GitHub Actions CI** (Workflow Run `#6` on PR `#33`).
- **Location:** `docs/conf.py` (initial commit `b870817` on branch `docs/sphinx-setup`).
- **Root Cause:** When PR #33 (*"Docs: configure Sphinx autodoc and generate API documentation"*) was opened, GitHub Actions failed the `Run Ruff tidy-code checker (MAIN-03)` step within 15 seconds because `import django` was grouped without a blank line after standard-library imports (`os`, `sys`) (`I001`), and `docs/conf.py` lacked a trailing newline (`W292`).
- **Corrective Action Taken:** Fixed immediately in follow-up commit **`27bd9e1`** (*"Docs: auto-fix import sorting and trailing newline in docs/conf.py"*), separating `import django` from `os`/`sys` and adding the trailing newline. Workflow Run `#7` passed both Ruff and Django tests before PR `#33` merged (`910504c`).
- **Screenshot (PR #33 Checklist & `27bd9e1` CI Fix):**

![Finding_1_PR33](https://github.com/user-attachments/assets/b208cdbd-7ac9-4126-8088-687017b733da)

---

### Finding 2 — Mobile Touch-Target (`44×44 px`) & Horizontal Overflow Regression Caught by CI
- **Tool:** **GitHub Actions CI (`python manage.py test`)** executing `accounts/test_phone_layout.py` (Workflow Run `#45424805962`).
- **Location:** `templates/base.html`.
- **Root Cause:** After a shared layout polish merge, the CSS rules enforcing `min-height: 44px; min-width: 44px;` on buttons (`COMP-02`) and `overflow-x: auto;` on wide tables (`SYS-02`) were accidentally dropped from `templates/base.html`, causing `PhoneFriendlyLayoutTests.test_base_template_has_viewport_and_44px_button_targets` to fail in CI.
- **Corrective Action Taken:** Fixed in commit **`964db4d`** (*"UI: restore mobile touch-target and overflow-x rules in base.html"*), restoring explicit `44px` minimum touch targets on `button, .btn, a.btn, input[type="submit"]` and `overflow-x: auto` on `.table-responsive, table`. Workflow Run `#45427123068` turned green immediately.
- **Screenshot (Commit `964db4d` Restoring `44px` Touch Targets & `overflow-x`):**

![Finding_2_Commit_964db4d](https://github.com/user-attachments/assets/bfa04cff-0b3f-4af1-b8c3-36835b4dc7f3)

---

### Finding 3 — Unused Imports (`F401`), Line-Length (`E501`), and CI Workflow YAML Fixes
- **Tool:** **Ruff (`ruff check .`)** & **GitHub Actions CI** when activating `.github/workflows/ci.yml` on `main`.
- **Location:** Multiple app modules and test files across `accounts/`, `booking/`, `queueing/`, and `scheduling/`.
- **Root Cause:** Initial full-repo Ruff runs flagged unused test imports (`F401`), unsorted import blocks (`I001`), lines exceeding 100 characters (`E501`), and missing docstrings (`D102`/`D103`).
- **Corrective Action Taken:** Resolved across the repository in commit **`CI: fix workflow YAML indentation and resolve all Ruff lint checks`** (`PR #22` / `main` CI activation), bringing `ruff check .` to **`All checks passed!`** with zero warnings across all subsequent workflow runs.
- **Screenshot (GitHub Actions CI Workflow Runs Passing):**

![Finding_3_Actions_CI](https://github.com/user-attachments/assets/8d2529e7-7e4a-489f-9824-78940e7732b2)

---

### Finding 4 — Self-Guardianship Loophole (`guardian=self`) Caught in `accounts/models.py`
- **Tool:** **GitHub Pull Request Review (`PR #4`)** + Django Model Validation.
- **Location:** `accounts/models.py` (`Patient` model).
- **Root Cause:** During early `AUTH-00` model review, a `Patient` record could reference itself in the `guardian` foreign key (`guardian = self`), which would create a circular reference in dependent lookups (`AUTH-04`).
- **Corrective Action Taken:** Fixed in **PR #4** (*"AUTH-00: block self-guardianship and limit guardian choices"* by `@munim456`), adding a PostgreSQL `CheckConstraint(check=~Q(id=F("guardian")), name="patient_cannot_be_own_guardian")` and restricting `limit_choices_to={"guardian__isnull": True}`.
- **Screenshot (GitHub PR #4 — Self-Guardianship Constraint Fix):**

![Finding_4_PR4](https://github.com/user-attachments/assets/26ac71f1-c4d8-4834-b03a-1aa80385cde7)

---

### Finding 5 — Cancelled Appointments Blocking Slot Rebooking Caught in `booking/models.py`
- **Tool:** **GitHub Pull Request Review (`PR #6`)** + `booking/tests.py`.
- **Location:** `booking/models.py` (`Appointment` model).
- **Root Cause:** An unconditional unique constraint on `(doctor, slot_start)` would satisfy `REL-02` (no double booking) but break `BR-01` by permanently locking a time slot even after a patient cancelled or was marked `No-Show`.
- **Corrective Action Taken:** Fixed in **PR #6** (*"BKG-00: add Appointment model with no-double-booking constraint"* by `@Arafat77-alt`), scoping the database constraint with `condition=Q(status__in=["booked", "checked_in", "in_consultation"])` (`uniq_active_slot_per_doctor`).
- **Screenshot (GitHub PR #6 — Conditional Slot Uniqueness Fix):**

![Finding_5_PR6](https://github.com/user-attachments/assets/b0afaa3f-5cb8-40ec-a2e7-44acdbbfd2e2)

---

### Finding 6 — Pylint (`W0718`, `C0415`, `W0613`) & Bandit (`CWE-259`) Security Audit
- **Tools:** **Pylint (`pylint --load-plugins=pylint_django`)** & **Bandit (`bandit -r .`)**.
- **Root Cause & Engineering Resolution:**
  1. **Pylint (`9.97/10`):** Flagged `W0718` (`broad-exception-caught`) in `notifications/services.py` (lines `104`, `115`, `145`, `163`, `193`) and `scheduling/leave.py:117`, plus `C0415` (`import-outside-toplevel` for `datetime.date`) in `accounts/views.py:155` and `W0613` (`unused-argument` on `_appointment_link(appointment)`) in `notifications/services.py:67`. Review verified that catching gateway exceptions in `notifications/services.py` is **required by SRS `REL-04`** so an SMS/email failure never aborts a confirmed booking or doctor leave, while `logger.debug(..., exc_info=True)` preserves stack traces.
  2. **Bandit Security Scan:** Across **`4,783` lines of production application code**, Bandit reported **`0` security issues** (`0 High`, `0 Medium`, `0 Low`). Across all `9,194` lines including tests, Bandit flagged `73` Low-severity `B105`/`B106` notices—100% synthetic test passwords (e.g., `"Strong-pass-123"` in `accounts/tests.py`, required by `PRIV-04`).
