# General Coding Rules

## **CRITICAL** Tooling Preferences

### Python

#### Prototype (local-first)

* **Packager & Runtime:** [uv](https://docs.astral.sh/uv/) ≥0.4

  * Setup: `uv sync`
  * Run: `uv run script.py`
* **Linter & Formatter:** [Ruff](https://docs.astral.sh/ruff/)

  * Lint: `uv run ruff check .`
  * Format: `uv run ruff check . --fix`
  * Ruff is the *single source of truth* for code style — **no Black**.
* **Type Checker:** [ty](https://github.com/zauberzeug/ty) — `uv run ty check src`

  * Enforce strict typing; missing or loose annotations fail CI.
* **Testing:** [pytest](https://docs.pytest.org/en/stable/) — `uv run pytest`

  * Prefer functional fixtures and composable helpers, not class-based tests.
* **Dependencies:**

  * Add: `uv add <package>`
  * Remove: `uv remove <package>`
  * Keep `pyproject.toml` and lockfile in sync via `uv sync`.

#### Production (managed)

* **Build & Deploy:** `uv build`; use `uv sync --locked` for deterministic environments.
* **Quality Gates:** CI must enforce `ruff`, `ty`, and test passes.
* **Interface Enforcement:**

  * Use `ruff` plugins or custom linter to ensure all ABCs have complete implementations.
* **Tests:**

  * Require ≥90% coverage (`pytest --cov --cov-fail-under=90`).
* **Logging & Observability:**

  * Default to `loguru` for structured logs.
  * Avoid monkeypatching or runtime overrides.

#### Bootstrapping

* **Baseline Workflow:**

  * `uv init` → `uv sync` → `uv run ruff check .` → `uv run ty check src` → `uv run pytest`
  * Commit only once all quality gates pass.
* **Framework Starter:**

  * For APIs, use [FastAPI](https://fastapi.tiangolo.com/) + [httpx](https://www.python-httpx.org/).
  * Prefer function-based routes and dependency injection; minimize class-based controllers.

---

## **CRITICAL** Must Follow Design Principles

### General Principles

* **ALWAYS** begin from a verified baseline (passing `ruff`, `ty`, `pytest`) before development.
* **ALWAYS** commit atomically with meaningful messages.
* **NEVER** maintain backward compatibility with deprecated code — remove legacy.
* **NEVER** create fallback mechanisms or “temporary hacks.”
* **ALWAYS** minimize complexity — Cyclomatic Complexity < 10.
* **ALWAYS** prefer established libraries over bespoke code.
* **ALWAYS** use descriptive symbol names; avoid suffixes like “Refactored” or “V2.”

---

### Composition-First Architecture

* Favor **pure functions** and **factory functions** over class hierarchies.
* Use **dependency injection** through function parameters or closures.
* Represent immutable entities with `@dataclass(frozen=True)` or `NamedTuple`.
* Compose behaviors with functions and modules — not inheritance.
* Use **Abstract Base Classes (ABCs)** or **`typing.Protocol`** for explicit contracts.
* Limit inheritance depth to 1 (interface → implementation).
* Define all interfaces under `src/interfaces/`.

---

### SOLID Principles — Pythonic Enforcement

#### **S — Single Responsibility**

Each module, function, or class must serve one clearly defined purpose.
Split logic when responsibilities diverge.

#### **O — Open/Closed**

Design abstractions so new behaviors are added via extension, not modification.
Base ABCs and Protocols must remain stable.

#### **L — Liskov Substitution**

Subclasses and Protocol implementations must behave consistently with their contracts.
Inputs, outputs, and exceptions must not change semantics.

#### **I — Interface Segregation**

Favor small, focused ABCs or Protocols.
Never force clients to depend on methods they don’t use.

#### **D — Dependency Inversion**

High-level modules depend only on abstractions.
Inject dependencies via parameters or factory composition roots.

---

### DRY Principles — Implementation Hygiene

* **ALWAYS** consolidate repeated behavior immediately.
* **NEVER** duplicate logic, even temporarily.
* **ALWAYS** centralize constants in `settings.py`.
* **NEVER** maintain multiple sources of truth.

---

## **CRITICAL** Must Follow Behavior Rules

### Research & Editing Discipline

* **ALWAYS** read the entire file or symbol before modification.
* **ALWAYS** understand imported dependencies before editing.
* **NEVER** modify code without reviewing context.

### Security Requirements

* **NEVER** commit or log secrets, API keys, or credentials.
* **ALWAYS** manage secrets via environment variables or secret stores.
* **NEVER** store sensitive data in plaintext.

### Prohibited Reward Hacking

* **NEVER** use placeholders or stubs outside test contexts.
* **NEVER** bypass linters, type checks, or tests.
* **NEVER** silence or hide failing code.
* **NEVER** use `--skip`, `--no-verify`, or similar flags.
* **NEVER** implement fallback or temporary workaround modes.
* **NEVER** move prohibited behavior to other modules.

---

## Personal Workflow Habits

* **Planning Loop:** Research → propose → await approval → implement → document decisions in `session-notes.md`.
* **Coding Loop:** Write → run `ruff`, `ty`, `pytest` → commit → push → review.
* **Documentation:** Maintain inline docstrings and project-level `README.md`.

---

## ExecPlan Protocol

* Instantiate from `~/.codex/execplan.md` when planning.
* Keep updated throughout execution — it is a living record.
* Acceptance criteria in **Success Criteria** define “done”; verify before completion.

---

## 🧱 Example: Composition + Interface Design

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Protocol

class PaymentProcessor(ABC):
    @abstractmethod
    def charge(self, amount: float) -> bool: ...

class Notifier(Protocol):
    def send(self, message: str) -> None: ...

@dataclass(frozen=True)
class Config:
    tax_rate: float

def make_billing_service(processor: PaymentProcessor, notifier: Notifier, config: Config):
    def bill(amount: float):
        total = amount * (1 + config.tax_rate)
        if processor.charge(total):
            notifier.send(f"Charged {total}")
    return bill
```

---

## Enforcement Targets

| Category                 | Enforcement                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **Type Completeness**    | All public symbols typed; `ty` strict mode enforced.         |
| **Interface Coverage**   | Every ABC/Protocol has at least one implementation.          |
| **Substitution Safety**  | Implementations must match method signatures and exceptions. |
| **Dependency Purity**    | Business logic imports only abstract types.                  |
| **Composition Validity** | Inheritance depth ≤ 1; prefer factories and closures.        |

---

# 🧩 Python Project Defaults

## Package & Environment Management

* Use `uv` for all package and environment management.
* Initialize all projects with `uv sync`.
* **Never** use `pip` or `poetry` unless explicitly requested.

## Dependencies & Libraries

### Logging

* Default to [loguru](https://github.com/Delgan/loguru) for structured logging.
* Add `loguru` to dependencies during project setup.
* Configure logging at startup; do not reconfigure globally.

### Settings Management

* Use [pydantic-settings](https://docs.pydantic.dev/latest/usage/pydantic_settings/) for configuration.
* Implement settings as an `@lru_cache` singleton pattern:

```python
from functools import lru_cache
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # settings fields here
    pass

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

## Code Quality

* Use `ruff` for all linting and formatting.
* Configure `ruff` in `pyproject.toml`.
* Run `ruff check` and `ruff format` before commits.

## Project Tooling

### Command Runner

* Use `Makefile` as the standard command runner.
* Include common targets:

  * `install`
  * `lint`
  * `format`
  * `test`
  * `clean`
  * `run`

### Environment Variables

* Use [direnv](https://direnv.net/) for environment variable management.
* Create `.envrc.custom` for custom environment setup.
* Activate the virtual environment within `.envrc.custom`:

  ```bash
  source .venv/bin/activate
  ```
* Add `.envrc.custom` to `.gitignore`.

## Project Structure

* Keep `README.md` concise — prefer declarative bullet points over narrative paragraphs.
* Use lightweight modules with clear separation by purpose (`interfaces/`, `services/`, `utils/`, etc.).
