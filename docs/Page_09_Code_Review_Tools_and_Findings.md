# 9. Code Review Tools and Findings

To enforce SRS Requirements **`MAIN-03`** (linter-checked coding standard), **`MAIN-04`** (pull-request peer review and passing automated CI), and **`SEC-07`** (security verification), the ClinicQ team combined **three automated static analysis tools** with **GitHub Pull Request reviews and GitHub Actions CI**.

---

## 9.1 Static Analysis & Code Review Tools Used

| Tool | Purpose in ClinicQ | Command Executed | Final Codebase Result |
| :--- | :--- | :--- | :--- |
| **1. Ruff** (`pyproject.toml` + GitHub Actions CI) | Enforces PEP 8 (`E`, `W`), Pyflakes bug checks (`F`), import sorting (`I`), and Google docstrings (`D`) on every push and PR. | `ruff check .` | **`All checks passed!`** (0 errors across all 7 apps & `docs/`) |
| **2. Pylint + `pylint-django`** | Deep AST static analysis for unused arguments, misplaced imports, method-override mismatches, exception handling, and duplicate code. | `pylint --load-plugins=pylint_django --ignore=migrations accounts booking consultation notifications queueing reports scheduling` | **Rated `9.80 / 10`** |
| **3. Bandit** (PyCQA Security Scanner) | Scans Python AST for security vulnerabilities (`CWE-259` hardcoded secrets, SQL injection, weak `random` PRNG, unsafe `eval`/subprocess). | `bandit -r accounts booking consultation notifications queueing reports scheduling config -x "*/tests*,*/test_*,*/migrations/*"` | **`0` issues in `4,783` lines of production code** (`0 High`, `0 Medium`, `0 Low`) |
| **4. GitHub Pull Requests & Actions CI** | Enforces PR checklist (`.github/pull_request_template.md`), peer review, and automated `ruff check .` + `python manage.py test` runs before/on merge. | `.github/workflows/ci.yml` across 33 PRs | **CI Passing** (Run `#45427123068` & PR `#33` Run `#8`) |

### Terminal & CI Verification Screenshots
- **Static Analysis CLI Execution (`ruff`, `bandit`, `pylint`):**  
  `[Upload Screenshot: 7_ruff_bandit_pylint_terminal_output.png]`
- **GitHub Actions CI Workflow History (Showing Real Failure Catches & Green Fix Runs):**  
  `[Upload Screenshot: 5_github_actions_ci_passing.png]`
- **Pull Request #33 Checklist & Automated CI Ruff Fix Commit (`27bd9e1`):**  
  `[Upload Screenshot: 6_github_pr33_ci_ruff_check.png]`

---

## 9.2 Six Real Findings, Root-Cause Explanations, and Corrective Actions

Below are **six genuine issues** detected by Ruff, GitHub Actions CI, Pylint, Bandit, and PR reviews on the ClinicQ repository, along with their root causes and the exact fixes or engineering decisions applied:

---

### Finding 1 — Unsorted Imports (`I001`) and Missing Trailing Newline (`W292`) Caught on PR #33
- **Tool:** **Ruff (`ruff check .`) in GitHub Actions CI** (Workflow Run `#6` on PR `#33`).
- **Location:** `docs/conf.py` (initial commit `b870817` on branch `docs/sphinx-setup`).
- **What Was Detected:** When PR #33 (*"Docs: configure Sphinx autodoc and generate API documentation"*) was opened, GitHub Actions failed the `Run Ruff tidy-code checker (MAIN-03)` step within 15 seconds because `import django` was grouped without a blank line after standard-library imports (`os`, `sys`) (`I001`), and `docs/conf.py` lacked a trailing newline (`W292`).
- **Corrective Action Taken:** Fixed immediately in follow-up commit **`27bd9e1`** (*"Docs: auto-fix import sorting and trailing newline in docs/conf.py"*), separating `import django` from `os`/`sys` and adding the trailing newline. Workflow Run `#7` passed both Ruff and Django tests before PR `#33` merged (`910504c`).

---

### Finding 2 — Mobile Touch-Target (`44×44 px`) & Horizontal Overflow Regression Caught by CI
- **Tool:** **GitHub Actions CI (`python manage.py test`)** executing `accounts/test_phone_layout.py` (Workflow Run `#45424805962` / Commit `edcdcc1`).
- **Location:** `templates/base.html`.
- **What Was Detected:** After a shared layout polish merge, the CSS rules enforcing `min-height: 44px; min-width: 44px;` on buttons (`COMP-02`) and `overflow-x: auto;` on wide tables (`SYS-02`) were accidentally dropped from `templates/base.html`, causing `PhoneFriendlyLayoutTests.test_base_template_has_viewport_and_44px_button_targets` to fail in CI.
- **Corrective Action Taken:** Fixed in commit **`964db4d`** (*"UI: restore mobile touch-target and overflow-x rules in base.html"*), restoring explicit `44px` minimum touch targets on `button, .btn, a.btn, input[type="submit"]` and `overflow-x: auto` on `.table-responsive, table`. Workflow Run `#45427123068` turned green immediately.

---

### Finding 3 — Unused Imports (`F401`), Line-Length (`E501`), and Docstring Gaps Across Merged Apps
- **Tool:** **Ruff (`ruff check .`)** when activating `.github/workflows/ci.yml` on `main`.
- **Location:** Multiple app modules and test files across `accounts/`, `booking/`, `queueing/`, and `scheduling/`.
- **What Was Detected:** Initial full-repo Ruff runs flagged unused test imports (`F401`), unsorted import blocks (`I001`), lines exceeding 100 characters (`E501`), and missing docstrings (`D102`/`D103`).
- **Corrective Action Taken:** Resolved across the repository in commit **`CI: fix workflow YAML indentation and resolve all Ruff lint checks`**, bringing `ruff check .` to **`All checks passed!`** with zero warnings.

---

### Finding 4 — Unused Function Parameter (`W0613`) and Function-Level Import (`C0415`)
- **Tool:** **Pylint (`pylint --load-plugins=pylint_django`)**.
- **Locations:**
  1. `notifications/services.py:67:22` — `W0613: Unused argument 'appointment' (unused-argument)` in `_appointment_link(appointment)`.
  2. `accounts/views.py:155:8` — `C0415: Import outside toplevel (datetime.date) (import-outside-toplevel)` inside `patient_register_verify()`.
- **Root Cause & Action:**
  - In `notifications/services.py`, `_appointment_link(appointment)` accepts `appointment` to match the helper signature of `_doctor_label(appointment)` and `_appointment_when(appointment)`, while linking to the unified `/booking/appointments/mine/` dashboard (`reverse('booking:my_appointments')`). Keeping the parameter preserves a uniform helper interface if per-appointment detail URLs are used later.
  - In `accounts/views.py:155`, `from datetime import date` was placed inside the `POST` branch of `patient_register_verify()` rather than at the top of `accounts/views.py`; Pylint flagged it for top-level module placement per PEP 8.

---

### Finding 5 — Broad Exception Catching (`W0718`) Audited Against SRS Reliability Rule `REL-04`
- **Tool:** **Pylint (`W0718: broad-exception-caught`)**.
- **Locations:** `notifications/services.py` (lines `104`, `115`, `145`, `163`, `193`) and `scheduling/leave.py:117`.
- **Root Cause & Engineering Resolution:** Pylint flagged `except Exception:` around SMS and email delivery calls:
  ```python
  # notifications/services.py (lines 101–106)
  try:
      if send_sms(mobile, message):
          return Notification.Channel.SMS, Notification.Status.SENT, attempts, mobile
  except Exception:
      logger.debug("SMS confirmation failed on attempt %s", attempts, exc_info=True)
  ```
  During review, we verified that this is **required by SRS `REL-04`** (*"If the SMS or email service fails, the main action (for example booking) shall still succeed. The system shall retry sending up to 3 times and show the message in the user's in-app inbox"*). Catching unexpected gateway exceptions prevents a third-party SMS/SMTP outage from rolling back a patient's confirmed appointment or doctor's leave, while `logger.debug(..., exc_info=True)` and `logger.exception(...)` preserve full stack traces in server logs instead of failing silently.

---

### Finding 6 — Security Scan (`Bandit CWE-259`) & Database Constraint Bugs Fixed in PR Reviews
- **Tools:** **Bandit (`bandit -r .`)** + **GitHub Pull Request Reviews (`PR #4`, `PR #6`)**.
- **What Was Detected:**
  1. **Bandit Scan:** Across **`4,783` lines of production application code**, Bandit reported **`0` security issues** (`0 High`, `0 Medium`, `0 Low`) — confirming that OTP generation uses Python's cryptographic `secrets` module rather than `random`, `SECRET_KEY` is loaded via `django-environ` rather than hardcoded in `config/settings.py`, and all queries use Django's parameterized ORM (`SEC-07`). When scanning all **`9,194` lines including tests**, Bandit flagged `73` Low-severity `B105`/`B106` (`hardcoded_password_funcarg`) notices, **100% of which were synthetic test passwords** (such as `"Strong-pass-123"` and `"Temp-pass-4821"` in `accounts/tests.py` and `scheduling/tests.py`, required by `PRIV-04` synthetic test data rules).
  2. **PR Review Logic Catches (`PR #4` & `PR #6`):** Peer review caught that a `Patient` could reference `guardian=self` (fixed in **PR #4** via `CheckConstraint(check=~Q(id=F("guardian")), name="patient_cannot_be_own_guardian")`) and that cancelled appointments would block rebooking the same slot unless the unique constraint filtered only active statuses (fixed in **PR #6** via `UniqueConstraint(..., condition=Q(status__in=ACTIVE_STATUSES))`).
