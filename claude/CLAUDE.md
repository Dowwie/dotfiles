# Python Development Standards

## Critical Tooling Stack

### Package & Runtime Management
- **Primary Tool:** `uv` (≥0.4) — NEVER use `pip` or `poetry` unless explicitly requested
- **Setup:** `uv sync`
- **Run Scripts:** `uv run script.py`
- **Add Dependencies:** `uv add <package>`
- **Remove Dependencies:** `uv remove <package>`

### Code Quality Tools
- **Linter & Formatter:** `ruff` — single source of truth for style
  - Lint: `uv run ruff check .`
  - Format: `uv run ruff check . --fix`
  - **NEVER** use Black or other formatters
- **Type Checker:** `ty` — enforce strict typing
  - Check: `uv run ty check src`
  - All public symbols must be typed
- **Testing:** `pytest`
  - Run: `uv run pytest`
  - Coverage requirement: ≥90% (`pytest --cov --cov-fail-under=90`)

### Default Libraries
- **Logging:** `loguru` — always add to new projects
- **Settings:** `pydantic-settings` with `@lru_cache` singleton pattern
- **API Framework:** FastAPI + httpx
- **Environment:** direnv with `.envrc.custom` (add to `.gitignore`)

## Mandatory Workflow

### Before ANY Code Changes
1. Read entire file/symbol before modification
2. Understand all imported dependencies
3. Verify baseline passes: `ruff` → `ty` → `pytest`

### Development Loop
1. Write code
2. Run `uv run ruff check . --fix`
3. Run `uv run ty check src`
4. Run `uv run pytest`
5. Only commit when all checks pass
6. Use atomic commits with meaningful messages

### Project Initialization
```bash
uv init
uv sync
uv add loguru pydantic-settings
# Setup Makefile, .envrc.custom, README.md
uv run ruff check .
uv run ty check src
uv run pytest
```

## Architecture Principles (NON-NEGOTIABLE)

### Composition-First Design
- **ALWAYS** prefer pure functions and factory functions over classes
- **ALWAYS** use dependency injection via parameters or closures
- **ALWAYS** use `@dataclass(frozen=True)` or `NamedTuple` for immutable data
- **NEVER** create deep inheritance hierarchies (max depth: 1)
- **ALWAYS** prefer `typing.Protocol` for interface contracts (structural subtyping)
- **ONLY** use ABC when shared implementation logic is required
- Define interfaces in `src/interfaces/` using `typing.Protocol` by default

### Constructor Patterns

**Classmethod constructors (prefer for class instances):**
```python
class Client:
    @classmethod
    def from_spec(cls, spec_path: str) -> "Client":
        """Validate before construction."""
        if not Path(spec_path).exists():
            raise FileNotFoundError(f"Not found: {spec_path}")
        return cls(...)  # Construct after validation

# Usage: No naming collision
from mylib import Client
client = Client.from_spec("spec.json")
```

**Factory functions (use for closures/functions, not class instances):**
```python
def make_processor(config: Config):
    """Return function, not class instance."""
    def process(data: str) -> bool:
        return process_with_config(data, config)
    return process
```

**When to use:**
- `@classmethod` → Creating class instances (from_spec, from_config, from_url)
- Factory function → Creating closures, returning functions
- Plain `__init__` → Simple initialization, no validation needed

### Protocol vs ABC: Decision Matrix

**Default Choice: Protocol** (PEP 544 structural subtyping)

```python
from typing import Protocol

class DataProcessor(Protocol):
    """Pure interface contract - structural subtyping."""
    def process(self, data: str) -> bool: ...
    def validate(self, data: str) -> bool: ...

# Any class with matching methods satisfies the protocol
class JSONProcessor:  # No inheritance needed!
    def process(self, data: str) -> bool:
        return True
    def validate(self, data: str) -> bool:
        return bool(data)

# Type checker validates compatibility
processor: DataProcessor = JSONProcessor()  # ✅ Valid
```

**When to Use Protocol:**
- ✅ Pure interface contracts (no shared implementation)
- ✅ Need duck typing with compile-time safety
- ✅ Want to avoid inheritance coupling
- ✅ Third-party classes should satisfy interface without modification
- ✅ Easier mocking (any object with matching signature works)

**When to Use ABC:**
- ⚠️ Shared implementation logic across subclasses (e.g., common `__init__`)
- ⚠️ Need runtime validation with `isinstance()` checks
- ⚠️ Enforcing method implementation at class definition time

```python
from abc import ABC, abstractmethod

class Stage(ABC):  # Use ABC only when sharing implementation
    def __init__(self, name: str):
        self.name = name  # Shared initialization logic

    @abstractmethod
    async def execute(self) -> None: ...

class ValidationStage(Stage):  # Inherits __init__
    async def execute(self) -> None:
        print(f"Executing {self.name}")
```

**Key Principles:**
- **Default to Protocol** unless you need shared implementation
- Protocols are compile-time only (type hints)
- ABCs enforce contracts at runtime (class definition)
- Use `@runtime_checkable` decorator if you need `isinstance()` with Protocols

### SOLID Enforcement
- **Single Responsibility:** Each module/function/class has ONE purpose
- **Open/Closed:** Extend via abstractions, not modifications
- **Liskov Substitution:** Implementations must honor contracts exactly
- **Interface Segregation:** Small, focused Protocols only
- **Dependency Inversion:** Business logic depends on Protocol abstractions only

### DRY Principles
- **ALWAYS** consolidate repeated code immediately
- **NEVER** duplicate logic, even temporarily
- **ALWAYS** centralize constants in `settings.py`

## Security & Quality Gates

### Security Requirements
- **NEVER** commit secrets, API keys, or credentials
- **ALWAYS** use environment variables or secret stores
- **NEVER** log sensitive data

### Prohibited Practices (REWARD HACKING)
- **NEVER** use placeholders/stubs outside tests
- **NEVER** bypass linters, type checks, or tests
- **NEVER** use `--skip`, `--no-verify` flags
- **NEVER** silence failing code
- **NEVER** implement fallback/workaround modes
- **NEVER** move prohibited behavior to other modules

## Code Complexity Rules
- Maximum Cyclomatic Complexity: 10
- Prefer established libraries over custom implementations
- Use descriptive names; avoid suffixes like "Refactored" or "V2"

## Project Structure Standards

### Command Runner
Use `Makefile` with these targets:
- `install` — setup project
- `lint` — run ruff
- `format` — auto-fix with ruff
- `test` — run pytest
- `clean` — remove artifacts
- `run` — start application

### Documentation
- Keep `README.md` concise with declarative bullet points
- Maintain inline docstrings (Google style)
- Document decisions in `session-notes.md`

### Directory Layout
```
src/
  interfaces/    # Protocols (and rare ABCs with shared implementation)
  utils/         # Helpers
tests/           # Test suite
```

## Settings Management Pattern

```python
from functools import lru_cache
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # fields here
    pass

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

## Composition Example Template

```python
from dataclasses import dataclass
from typing import Protocol

# Protocol-first interfaces (structural subtyping)
class DataProcessor(Protocol):
    """Process data - no inheritance required."""
    def process(self, data: str) -> bool: ...

class Notifier(Protocol):
    """Send notifications - structural contract."""
    def notify(self, message: str) -> None: ...

# Immutable configuration
@dataclass(frozen=True)
class Config:
    setting: str

# Factory function with dependency injection
def make_service(processor: DataProcessor, notifier: Notifier, config: Config):
    """Create service using injected dependencies."""
    def execute(data: str) -> None:
        if processor.process(data):
            notifier.notify(f"Processed with {config.setting}: {data}")
    return execute

# Any class with matching methods satisfies the protocol
class JSONProcessor:  # No inheritance!
    def process(self, data: str) -> bool:
        return bool(data)

class EmailNotifier:  # No inheritance!
    def notify(self, message: str) -> None:
        print(f"Email: {message}")

# Compose at runtime
config = Config(setting="strict")
service = make_service(JSONProcessor(), EmailNotifier(), config)
service("test data")
```

## Personal Workflow Notes
- Planning: Research → Propose → Await Approval → Implement → Document
- Use ExecPlan from `~/.codex/execplan.md` for planning
- Keep ExecPlan updated throughout execution
- Verify Success Criteria before marking tasks complete

## Testing Standards
- Prefer functional fixtures and composable helpers
- Avoid class-based tests
- Follow AAA pattern: Arrange, Act, Assert
- Test coverage must be ≥90%

## Enforcement Checklist
Before any commit, verify:
- [ ] All public symbols have type annotations
- [ ] `ty check` passes in strict mode
- [ ] Interfaces use Protocol (ABCs only if shared implementation needed)
- [ ] All Protocols have concrete implementations
- [ ] No inheritance depth > 1
- [ ] Business logic imports only Protocol abstractions
- [ ] `ruff check` passes with no errors
- [ ] `pytest` passes with ≥90% coverage
- [ ] No secrets in code or logs
- [ ] Cyclomatic complexity < 10
- [ ] No duplicate code
