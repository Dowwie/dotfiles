# Python Project Defaults

## Package & Environment Management
- Use `uv` for all Python package and environment management
- Initialize projects with `uv sync`
- Never use `pip` or `poetry` unless explicitly requested

## Dependencies & Libraries

### Logging
- Default to `loguru` for all new Python projects
- Add `loguru` to dependencies during project setup

### Settings Management
- Use `pydantic-settings` for configuration management
- Implement settings as `@lru_cache` singleton pattern:
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
- Use `ruff` for linting and formatting
- Configure `ruff` in `pyproject.toml`
- Run `ruff check` and `ruff format` before commits


## Project Tooling

### Command Runner
- Use `Makefile` as the standard command runner
- Include common targets: `install`, `lint`, `format`, `test`, `clean`

### Environment Variables
- Use `direnv` for environment variable management
- Create `.envrc.custom` file for custom environment setup
- Activate virtual environment in `.envrc.custom`:
```bash
source .venv/bin/activate
```
- Add `.envrc.custom` to `.gitignore`

## Project Structure
- Keep CLAUDE.md files concise to minimize token usage
- Use declarative bullet points over narrative paragraphs
