# 6. Source Code Documentation Tool

To satisfy SRS Requirement **`MAIN-03`** (*automated source code documentation with $\ge 80\%$ public class and function docstring coverage*), ClinicQ uses **Sphinx** with the **Read the Docs (`sphinx_rtd_theme`)** HTML theme. Sphinx is configured in the repository's `docs/` directory (merged in **PR #33**) and extracts documentation directly from our Django models, views, forms, and service modules.

---

## 6.1 Installation and Setup Instructions

### 1. Installing Sphinx and the Read-the-Docs Theme
Inside the project environment (or Docker container), install Sphinx and `sphinx-rtd-theme`:
```bash
pip install sphinx sphinx-rtd-theme
```

### 2. Sphinx & Django Configuration (`docs/conf.py`)
Because ClinicQ is a Django 5.0 project, Sphinx must initialize Django (`django.setup()`) before `autodoc` imports models and views. Below is our actual `docs/conf.py` from the repository:

```python
# docs/conf.py
import os
import sys

import django

sys.path.insert(0, os.path.abspath(".."))
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "config.settings")
os.environ.setdefault("SECRET_KEY", "sphinx-docs-build-key")
os.environ.setdefault("DEBUG", "True")
os.environ.setdefault("DATABASE_URL", "sqlite:///db.sqlite3")
django.setup()

project = "ClinicQ"
copyright = "2026, Arafat, Munim, Fahim, Ishmam"
author = "Arafat, Munim, Fahim, Ishmam"
release = "1.0"

extensions = [
    "sphinx.ext.autodoc",
    "sphinx.ext.napoleon",
    "sphinx.ext.viewcode",
]

templates_path = ["_templates"]
exclude_patterns = ["_build", "Thumbs.db", ".DS_Store"]
html_theme = "sphinx_rtd_theme"
html_static_path = ["_static"]
```

Key extensions enabled:
- **`sphinx.ext.autodoc`** — Imports all 7 ClinicQ apps (`accounts`, `booking`, `consultation`, `notifications`, `queueing`, `reports`, `scheduling`) defined in `docs/api/*.rst` and extracts class, method, and function signatures.
- **`sphinx.ext.napoleon`** — Parses **Google-style** docstring sections (`Args:`, `Returns:`, `Raises:`, `Attributes:`) into structured HTML parameter tables.
- **`sphinx.ext.viewcode`** — Adds clickable **`[source]`** links beside every documented class and function, rendering syntax-highlighted source pages under `_modules/`.

### 3. Building the HTML Documentation
From the repository root (locally or inside Docker):
```bash
# Inside Docker:
docker compose exec web sphinx-build -b html docs docs/_build/html

# Or locally with Python 3.12:
sphinx-build -b html docs docs/_build/html
```
The generated static site is written to `docs/_build/html/index.html` and can be opened directly in Google Chrome.

---

## 6.2 Documentation Format and Conventions

Every public module, model, view, and service function in ClinicQ follows the **Google Python Style Guide** docstring convention:
1. **Imperative one-line summary** stating the responsibility of the function or class.
2. **SRS Traceability tags** naming the User Story ID (`QUE-08`, `AUTH-01`, `SCH-01`) and Business Rule / Non-Functional Requirement ID (`BR-10`, `BR-12`, `SEC-04`).
3. **`Args:` / `Attributes:`** listing every input parameter or model field in plain language.
4. **`Returns:`** specifying the returned value or data structure.
5. **`Raises:`** documenting `ValidationError` conditions and plain-language `USAB-06` refusal cases.

---

## 6.3 Sample Source Code with Documentation Comments

Below is a real service function from **`queueing/completion.py`** (merged in **PR #27**) demonstrating our Google-style docstring format:

```python
# queueing/completion.py
def complete_consultation(entry, now=None):
    """Finish a doctor's active consultation and record its duration (QUE-08).

    Marks the queue entry as Completed, records completed_at (setting
    started_at from called_at or now if not already set), and updates the
    linked appointment status to Completed.

    Args:
        entry: The active QueueEntry (state Called or In consultation).
        now: Optional timezone-aware completion time. Defaults to current time.

    Returns:
        The consultation duration in minutes as a non-negative float.

    Raises:
        ValidationError: With a plain-language message if the patient is not
            currently called or in consultation.
    """
    if entry.state not in ACTIVE_STATES:
        raise ValidationError(
            "Only a patient who has been called or is in consultation can be "
            "marked as completed. Call the patient first."
        )
    current_time = now or timezone.now()
    with transaction.atomic():
        if entry.started_at is None:
            entry.started_at = entry.called_at or current_time
        entry.completed_at = current_time
        entry.state = QueueEntry.State.COMPLETED
        entry.save(update_fields=["started_at", "completed_at", "state"])

        appointment = entry.appointment
        appointment.status = Appointment.Status.COMPLETED
        appointment.save(update_fields=["status", "updated_at"])

    return consultation_duration_minutes(entry)
```

---

## 6.4 Generated Documentation Output (Screenshots)

1. **Sphinx Documentation Home Page (`docs/_build/html/index.html`):**  
   *Shows the main table of contents, search bar, and module index generated with `sphinx_rtd_theme`.*

<img width="1280" height="820" alt="1_sphinx_home_index" src="https://github.com/user-attachments/assets/8f56e6cb-e301-45e0-a4c0-e9f338ff7aa6" />

2. **Application Package & Module Index (`docs/_build/html/api/modules.html`):**  
   *Displays all 7 ClinicQ Django packages (`accounts`, `booking`, `consultation`, `notifications`, `queueing`, `reports`, `scheduling`) and their submodules.*

<img width="1280" height="900" alt="2_sphinx_api_modules" src="https://github.com/user-attachments/assets/ca27279b-514a-45e1-ac1a-f3983860e841" />

3. **Rendered Service API Documentation (`docs/_build/html/api/accounts.services.html`):**  
   *Shows how `autodoc` and `napoleon` render `Parameters`, `Returns`, `Return type`, SRS Story/Requirement tags (`AUTH-01`, `SEC-04`), and `[source]` links.*

<img width="1280" height="950" alt="3_sphinx_accounts_services" src="https://github.com/user-attachments/assets/01a45f99-3817-4010-b87d-7834b0b8d889" />

4. **Syntax-Highlighted Source Code View (`docs/_build/html/_modules/queueing/completion.html`):**  
   *Generated by `sphinx.ext.viewcode` when clicking `[source]` on `complete_consultation` (`QUE-08`, `BR-10`), with `[docs]` back-links.*

<img width="1280" height="950" alt="4_sphinx_source_viewcode" src="https://github.com/user-attachments/assets/2de17783-129c-4290-b008-6bfeac965540" />
