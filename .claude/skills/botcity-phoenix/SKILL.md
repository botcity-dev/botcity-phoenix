---
name: botcity-phoenix
description: >
  Use this skill when the user asks to convert, migrate, transcribe, or port a UiPath
  automation to Python/BotCity — including when they mention a .xaml file, want to "move
  away from UiPath," "switch to BotCity," or generate a BotCity project from an existing
  workflow. Covers cloning the BeAPro template, summarizing the .xaml, transcribing each
  action to Python with BotCity Maestro SDK patterns, organizing files by responsibility,
  and generating an unpinned requirements.txt. Also trigger when a .xaml file is present
  and the user's intent is to run it as Python RPA, even if "BotCity" is not mentioned.
---

# UiPath → BotCity (Python) Migration Skill

Follow these four steps **in order**. Do not skip any step or write Python code before the user validates the `.xaml` summary.

1. **Clone the BeAPro template** as the project starting point.
2. **Read and summarize the `.xaml`** clearly and concisely.
3. **Transcribe the `.xaml` to Python**, action by action, using `references/uipath_to_python.md` and the Maestro SDK.
4. **Generate the folder structure** and an unpinned `requirements.txt`.
5. **Do not execute any test without user's specific request.**
---

## Step 1 — Clone the BeAPro Template

```bash
git clone https://github.com/botcity-dev/beapro-framework.git <project-name>
cd <project-name>
rm -rf .git
git init
```

After cloning:
- Rename the internal module folder to a `snake_case` name matching the automation domain (e.g., `sales_report`, `bank_reconciliation`).
- Update `README.md` with the project name.
- **Confirm the `bot_id` with the user before proceeding**.This name appears in Maestro and in logs.

If cloning fails (e.g., corporate network), offer this fallback:

```bash
python -m cookiecutter https://github.com/botcity-dev/bot-python-template
```

Minimum generated structure:

```
<bot_id>/
├── README.md
├── <bot_id>/
│   ├── __init__.py
│   ├── __main__.py     
│   ├── flow.py
│   └── resources/
├── requirements.txt
├── bot.py            # entrypoint — do not modify
├── build.sh
└── build.bat
```

Expand this structure in Step 4 - do not replace it.

---

## Step 2 — Read and Summarize the `.xaml`

Before transcribing, read the `.xaml` file and produce a **short prose summary** with:

1. **Automation objective** — one sentence.
2. **Main block sequence** — numbered list of up to 8–12 items, naming each UiPath action and what it does in context (e.g., *"3. `Open Browser` → opens the vendor portal"*).
3. **Inputs and outputs** — `in` arguments, files read, Orchestrator data consumed (assets, queue items), and what the automation produces.
4. **Points of attention** — `RetryScope`, `TryCatch`, conditional waits, external integrations, credential usage.

Focus on flow, not XML structure. If the `.xaml` references other `.xaml` files via `Invoke Workflow File`, list them and ask whether to open them too.

**Stop here and ask the user to validate the summary before proceeding to Step 3.**

---

## Step 3 — Transcribe to Python + BotCity SDK

With the validated summary, transcribe action by action. Read `references/uipath_to_python.md` before writing any code — it is the source of truth for library choices.

### Mandatory coding rules

- **PEP 8**: `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants.
- **Type hints** (PEP 484/604), `pathlib` instead of `os.path`, f-strings, `int | None` union syntax (Python 3.10+).
- **Docstrings** on all public functions and classes (Google or NumPy style). Inline comments only where intent is non-obvious.
- **Specific `try/except`** — never bare `except:` or `except Exception:` inside critical actions. Catch the most specific exception the library raises.
- **`logging` instead of `print`** — one logger per module via `logging.getLogger(__name__)`.
- **No hardcoded credentials** — use `os.environ` or `maestro.get_credential(...)`.
- **Idempotency** — the automation must be safe to run twice without unintended side effects.

### BotCity Maestro integration

Every corporate automation must, at minimum:
1. Instantiate the SDK and retrieve the current execution.
2. Report a final status (`SUCCESS`, `FAILED`, `PARTIALLY_COMPLETED`). Use the `AutomationTaskFinishStatus` Class.
3. Emit alerts (`AlertType.INFO/WARN/ERROR`) at critical points.
4. Log quantitative results (`new_result_step`) — items processed, items failed, etc.

Minimum `bot.py` skeleton:

```python
"""Bot entrypoint. Orchestrates the flow and reports to Maestro."""
from __future__ import annotations

import logging

from botcity.maestro import AlertType, BotExecution, BotMaestroSDK, AutomationTaskFinishStatus

BotMaestroSDK.RAISE_NOT_CONNECTED = False  # don't crash when running locally without Maestro

logger = logging.getLogger(__name__)


def main() -> None:
    """Entry point called by the BotCity Runner."""
    maestro = BotMaestroSDK.from_sys_args()
    execution: BotExecution = maestro.get_execution()

    try:
        from sales_report.flow import run_flow

        total, errors = run_flow(maestro, execution)

        maestro.finish_task(
            task_id=execution.task_id,
            status=AutomationTaskFinishStatus.SUCCESS,
            message=f"Processed {total} items, {errors} with errors.",
            total_items=total,
            processed_items=total - errors,
            failed_items=errors,
        )
    except Exception as exc:
        logger.exception("Unhandled failure during bot execution.")
        maestro.alert(
            task_id=execution.task_id,
            title="Critical failure",
            message=str(exc),
            alert_type=AlertType.ERROR,
        )
        maestro.finish_task(
            task_id=execution.task_id,
            status=AutomationTaskFinishStatus.FAILED,
            message=f"Error: {exc}",
        )
        raise


if __name__ == "__main__":
    main()
```

The single top-level `except Exception` in `main` is the only tolerated broad catch — it guarantees Maestro always receives a final status. All other exceptions must be handled close to the action that can fail.

### Library selection defaults

Consult `references/uipath_to_python.md` for each identified activity. Quick rules:

| Need | Default | Alternative |
|------|---------|-------------|
| Web automation | `botcity-web-framework` | `selenium` for existing projects |
| Desktop (Computer Vision) | `botcity-framework-core` | — |
| Excel | `openpyxl` for raw files | `pandas` for transformations |
| PDF text/tables | `pdfplumber` | `pypdf` ≥ 6.7.2 |
| HTTP/API | `httpx` | `requests` for legacy code |
| Database | `sqlalchemy` | — |
| OCR | `pytesseract` | `easyocr` for high volume |

Do not mix automation styles: if you choose `botcity-web-framework`, use it throughout. Never combine `selenium` and `playwright` in the same bot.

---

## Step 4 — Folder Structure and `requirements.txt`

Expand the BeAPro template into this layout:

```
<bot_id>/
├── bot.py                      # entrypoint + Maestro integration
├── README.md
├── requirements.txt
├── .env.example
├── <bot_id>/
│   ├── __init__.py
│   ├── __main__.py
│   ├── flow.py                     # high-level flow orchestration
│   ├── config.py                   # constants, paths, .env loading
│   ├── actions/                    # reusable atomic actions
│   │   ├── __init__.py
│   │   ├── web.py
│   │   ├── excel.py
│   │   ├── pdf.py
│   │   └── email_actions.py
│   ├── services/                   # external system integrations
│   │   ├── __init__.py
│   │   ├── api_client.py
│   │   └── database.py
│   ├── models/                     # dataclasses / Pydantic models
│   │   ├── __init__.py
│   │   └── entities.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── logger.py
│   │   └── retry.py
│   └── resources/
└── tests/
    ├── __init__.py
    ├── test_actions.py
    └── test_flow.py
```

Dependency flow always goes top-down: `bot.py` → `flow.py` → `actions/` + `services/`. Nothing in `actions/` or `services/` may import from `flow.py` or `bot.py`. Pass `maestro` and `execution` as parameters into `flow.py` — do not instantiate them there.

### `requirements.txt` — no pinned versions

List only the direct dependencies actually used, without pinning. The user will pin after running tests with `pip freeze` or `pip-compile`.

```text
# Core BotCity
botcity-framework-core
botcity-framework-web
botcity-maestro-sdk

# Add only what the transcription uses:
# playwright
# selenium
# pandas
# openpyxl
# pypdf
# pdfplumber
# httpx
# sqlalchemy
# python-dotenv
```

Add a comment when the reason isn't obvious:

```text
pypdf  # pin to >= 6.7.2 after tests (earlier versions have a DoS CVE)
```

---

## Gotchas

- **`bot_id` naming**: the `bot_id` must match exactly between the Maestro registration and the Python package name, a mismatch causes the Runner to fail silently.
- **`BotMaestroSDK.RAISE_NOT_CONNECTED = False`**: set this before any SDK call, not just before `from_sys_args()`, or local runs without a Maestro connection will crash immediately.
- **`Invoke Workflow File` in `.xaml`**: each referenced sub-workflow should become its own module in `flow.py`, not a separate `bot.py`. Ask the user for all referenced `.xaml` files before starting the transcription.
- **Selenium Grid vs. local WebDriver**: if the original UiPath bot targeted a remote browser, the `playwright` equivalent needs `connect_over_cdp()` or `connect()`, not a local `launch()`.
- **Assets vs. Arguments**: UiPath `in` arguments map to Maestro `execution.parameters`, not `maestro.get_credential()`. Credentials (user/password) go through `get_credential()`; config values go through `parameters`.

---

## Final Checklist

Before marking the project complete:

- [ ] BeAPro template cloned, `.git` removed, new `git init` done.
- [ ] `.xaml` summary validated by the user.
- [ ] Every `.xaml` activity has a Python equivalent (or is explicitly marked N/A with justification).
- [ ] `bot.py` reports a final status to Maestro on all code paths.
- [ ] `flow.py` reads like a high-level automation script.
- [ ] All functions have docstrings; all types are annotated.
- [ ] Specific `try/except` throughout; no bare `except:`.
- [ ] No hardcoded credentials — all via `.env` or Maestro credentials.
- [ ] Folder structure matches the layout above.
- [ ] `requirements.txt` is unpinned and contains only what is actually used.
- [ ] `README.md` explains how to run locally (`python -m <bot_id>`) and how to deploy it into Maestro(`./build.sh`).

---

## References

- `references/uipath_to_python.md` — full UiPath → Python/BotCity equivalence table. **Read before coding.**
- BotCity Maestro SDK docs: https://documentation.botcity.dev/maestro/maestro-sdk/
- BotCity Python template: https://github.com/botcity-dev/bot-python-template
- BeAPro framework: https://github.com/botcity-dev/beapro-framework
