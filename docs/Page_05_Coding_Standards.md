# 5. Coding Standards and Guidelines

To satisfy SRS Maintainability Requirement **`MAIN-03`** (*"Code shall follow a written coding standard checked by a linter. Every public class and function shall have a documentation comment with coverage of at least 80%"*) and Constraint **`C-06`**, the ClinicQ team follows **PEP 8**, **Django Coding Style Conventions**, and **Google-Style Python Docstrings**, enforced automatically on every push and pull request by **Ruff** in GitHub Actions CI.

---

## 5.1 Naming Conventions (PEP 8)

All identifiers across ClinicQ follow standard PEP 8 casing rules so any team member can distinguish classes, functions, variables, and configuration constants immediately during code review:

| Construct | Convention | Real ClinicQ Examples |
| :--- | :--- | :--- |
| **Django Apps & Modules** | `lowercase_snake_case` | `accounts`, `scheduling`, `booking`, `queueing`, `patient_booking.py`, `live_views.py` |
| **Models, Forms & Views (Classes)** | `PascalCase` | `User`, `Patient`, `Doctor`, `Appointment`, `QueueEntry`, `DoctorProfileForm` |
| **Functions, Methods & Variables** | `snake_case` | `add_to_queue()`, `complete_consultation()`, `slot_windows()`, `must_change_password` |
| **Module-Level Constants** | `UPPER_SNAKE_CASE` | `MAX_SKIPS = 2`, `GRACE_MINUTES = 15`, `BOOKING_WINDOW_DAYS = 14`, `MAX_FAILED_LOGINS = 5` |
| **Internal Module Helpers** | `_leading_underscore` | `_authorized_queue_entry()`, `_deliver()`, `_estimate_text_for_state()` |

**Real Code Example (`queueing/models.py` & `queueing/services.py`):**
```python
# Constants in UPPER_SNAKE_CASE (queueing/models.py)
MAX_SKIPS = 2
ACTIVE_STATES = ["called", "in_consultation"]


# Model classes and TextChoices enums in PascalCase (queueing/models.py)
class QueueEntry(models.Model):
    class State(models.TextChoices):
        WAITING = "waiting", "Waiting"
        CALLED = "called", "Called"
        IN_CONSULTATION = "in_consultation", "In consultation"
        RECEPTION_TO_DECIDE = "reception_to_decide", "Reception to decide"
        COMPLETED = "completed", "Completed"
        REMOVED = "removed", "Removed"


# Functions and local variables in snake_case (queueing/services.py)
def next_token_number(doctor, queue_date):
    """Return the next token number for a doctor on a day (BR-14)."""
    highest = QueueEntry.objects.filter(doctor=doctor, queue_date=queue_date).aggregate(
        highest=Max("token_number")
    )["highest"]
    return (highest or 0) + 1
```

---

## 5.2 File and Django App Layout (Separation of Concerns — `MAIN-01`)

To satisfy **`MAIN-01`** (*keeping business rules separate from screens and database code*) and prevent Git merge conflicts across 4 developers, each of our 7 Django apps (`accounts`, `scheduling`, `booking`, `queueing`, `consultation`, `notifications`, `reports`) splits responsibilities into dedicated files:

- **`models.py`** — Database schema, field validators, and PostgreSQL integrity constraints (`UniqueConstraint`, `CheckConstraint`).
- **`services.py` / Domain Service Modules (`estimate.py`, `completion.py`, `checkin.py`, `leave.py`, `patient_booking.py`)** — Pure Python business logic enforcing `BR-01` through `BR-16`. Views never implement business calculations inline.
- **`forms.py` / `profile_forms.py` / `reception_forms.py`** — Input validation and widget formatting.
- **`views.py` / `profile_views.py` / `live_views.py`** — Thin HTTP controllers handling authentication (`@role_required`, `@patient_required`), calling service functions, and rendering templates.
- **`urls.py`** — App-namespaced URL patterns (`app_name = "scheduling"`, `app_name = "queueing"`).
- **`tests.py` & `test_<feature>.py`** — Feature-scoped unit and integration test suites (`test_estimate.py`, `test_completion.py`, `test_live.py`, `test_profiles.py`).

**Real Code Example — Thin View Delegating to Service Layer (`queueing/live_views.py`):**
```python
@patient_required
def live_status(request, appointment_id):
    """Show the patient's token, queue position and live waiting estimate (QUE-04)."""
    entry = _authorized_queue_entry(request, appointment_id)
    if entry is None:
        return render(request, "accounts/forbidden.html", status=403)
    status = build_live_queue_status(entry)
    return render(
        request,
        "queueing/reception/live_status.html",
        {"entry": entry, "status": status},
    )
```

---

## 5.3 Module, Class, and Function Docstrings (`MAIN-03` Traceability)

Every module, model, form, view, and service function includes a triple-quoted **Google-style docstring** (`"""..."""`) that:
1. Summarizes what the object does in plain English on the first line.
2. Explicitly cites the **SRS Story ID** (e.g., `QUE-08`, `SCH-02`, `BKG-03`) and **Business Rule / NFR ID** (e.g., `BR-10`, `REL-02`, `SEC-04`) for bidirectional traceability.
3. Documents `Args:`, `Returns:`, `Raises:`, and model `Attributes:` so Sphinx `autodoc` + `napoleon` can generate API pages automatically.

**Real Code Example (`booking/patient_booking.py`):**
```python
def book_online_appointment(
    guardian_patient,
    target_patient_id,
    doctor,
    slot_start,
    booked_by,
    reason="",
    now=None,
):
    """Book an online appointment for a patient or their dependent (BKG-03).

    Enforces BR-01 (one active booking per slot), BR-03 (14-day window and
    30-minute lead time), REL-02 (atomic database uniqueness), and SEC-02
    (patient may only book for themselves or their own dependents).

    Args:
        guardian_patient: The logged-in Patient profile.
        target_patient_id: Primary key of the Patient receiving care.
        doctor: The active Doctor being booked.
        slot_start: Timezone-aware start datetime of the chosen slot.
        booked_by: The logged-in patient User account.
        reason: Optional visit reason.
        now: Optional current datetime for deterministic tests.

    Returns:
        Appointment: The newly created online Appointment.

    Raises:
        ValidationError: If the target patient does not belong to this family,
            the slot is no longer free, or someone else booked it first.
    """
```

---

## 5.4 Inline Comments, Plain-Language Errors (`USAB-06`), and Database Constraints (`REL-02`)

1. **Inline Comments Explain *Why*, Not *What*:** Comments state the exact SRS rule or edge case rather than restating obvious Python syntax.
2. **Plain-Language Validation Errors (`USAB-06`):** Every error message states what went wrong and what the user should do next without technical jargon or stack codes.
3. **Database-Enforced Invariants (`REL-02`):** Critical invariants (slot uniqueness, token uniqueness, max 2 skips, non-self guardianship) are written as PostgreSQL `UniqueConstraint` and `CheckConstraint` rules in `class Meta`.

**Real Code Example (`booking/models.py`):**
```python
# BR-01: only these statuses occupy a slot.
ACTIVE_STATUSES = ["booked", "checked_in", "in_consultation"]
```
```python
    class Meta:
        ordering = ["slot_start"]
        constraints = [
            models.UniqueConstraint(
                fields=["doctor", "slot_start"],
                condition=Q(status__in=ACTIVE_STATUSES),
                name="uniq_active_slot_per_doctor",
                violation_error_message=(
                    "This doctor already has an active appointment at this time. "
                    "A time slot can hold only one appointment. "
                    "Choose a different time or a different doctor, "
                    "or cancel the existing appointment first."
                ),
            ),
        ]
```

---

## 5.5 Import Ordering, Line Formatting, and Automated Enforcement (`Ruff`)

ClinicQ enforces formatting and lint rules via **`pyproject.toml`** and `.github/workflows/ci.yml`:
- **Line Length:** Maximum **100 characters** per line (4-space indentation, UTF-8 encoding, no tabs).
- **Import Sorting (`I` / isort):** Grouped into (1) Python standard library, (2) third-party / Django packages, and (3) local ClinicQ apps, separated by one blank line.
- **Enabled Rule Sets:** `E` & `W` (PEP 8 pycodestyle), `F` (Pyflakes unused imports/variables), `I` (import sorting), and `D` (pydocstyle Google convention, ignoring `D100`/`D104`/`D105`/`D106` only on boilerplate `__init__.py`, magic methods, and nested `Meta` classes).

**Real Configuration (`pyproject.toml`):**
```toml
[tool.ruff]
line-length = 100
target-version = "py312"
exclude = ["*/migrations/*"]

[tool.ruff.lint]
select = ["E", "W", "F", "I", "D"]
ignore = ["D100", "D104", "D105", "D106"]

[tool.ruff.lint.pydocstyle]
convention = "google"
```
