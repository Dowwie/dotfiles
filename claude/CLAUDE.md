# General Guidelines

## Comment Guidelines (NON-NEGOTIABLE)

### What NOT to Comment
**NEVER** write self-documenting comments that merely restate what the code does. If someone reading the code can understand the intent from the code itself, the comment is redundant.

### What TO Comment
Comments should explain **why**, not **what**. Add comments only when:
- The code's purpose or intent isn't obvious from the code itself
- There's a non-obvious reason for a design decision
- There's a workaround for a known issue or bug
- Complex business logic requires domain context
- Performance optimizations need justification

### Step Labels
**Avoid** inline step labels like `// Step 1:`, `// Step 2:`, etc. If the function is complex enough to need steps, break it into smaller functions with descriptive names or use the docstring to document the workflow.

### Section Comments
**Avoid** decorative section separator comments. If code needs logical grouping, extract it into a well-named function instead.

## Architecture Principles (NON-NEGOTIABLE)

### Architecture Design Heuristics
- Only include components explicitly required by the stated requirements
- If a capability isn't mentioned, don't design for it
- Ask before adding scope ("Should this system also handle X?")
- Domain models exist only for data crossing component boundaries
- Use interface abstractions where it makes sense; do not over-engineer their use
- Each interface: 2-5 methods typical; more suggests conflated responsibilities

### DRY Principles
- **ALWAYS** consolidate repeated code immediately
- **NEVER** duplicate logic, even temporarily
- **ALWAYS** centralize constants in configuration

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

## Documentation
- Keep `README.md` concise with declarative bullet points
- Maintain inline docstrings
- Document decisions in `session-notes.md`

## Commit Rules
- Do not mention attribution lines in commit messages
- NEVER, ever, **EVER** use Claude icons or emojis
- No secrets in code or logs
- No duplicate code
