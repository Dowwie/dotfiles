---
paths:
  - "**/*.ex"
  - "**/*.exs"
  - "**/mix.exs"
---

# Elixir Preferences

## Data Access

- **Prefer Access syntax (`opts[:key]`) over `Map.get/2` or `Keyword.get/2`** — Avoids locking into specific data structures
- **Use `map.key` for required keys, `map[:key]` for optional keys** — Static access fails fast on missing keys; dynamic access returns `nil`
- **Pattern match to assert structure** — Use `%User{} = user` before accessing fields for compile-time safety
- **Never use map access syntax (`struct[:field]`) on structs** — Structs do not implement Access by default. Use `struct.field` or `Ecto.Changeset.get_field/2`
- **List access** — Elixir lists do not support index-based access (`list[i]`). Use `Enum.at(list, i)`

## Error Handling

### Result Tuples and Control Flow

- **Don't pipe `{:ok, _} | {:error, _}` tuples through functions** — Use `with` or `case` to handle results directly
- **Keep error handling in the calling function** — The caller has the context to decide how to handle errors
- **Only return error tuples when callers can act on them** — If there's nothing the caller can do, raise or let it crash
- **Use `!` functions when you expect valid data** — Prefer `Jason.decode!` over `Jason.decode` when errors indicate bugs

### Using `with` Effectively

- **Avoid complex `else` clauses in `with`** — Don't use `else` to handle multiple specific error types; use `case` instead
- **Never tag `with` clauses for error differentiation** — If you're wrapping calls like `{:service, call_service()}`, use `case` instead
- **Omit `else` to bubble up errors** — When you only care about success, let non-matching results pass through
- **Normalize errors close to where they occur** — Wrap third-party errors in your own error types in helper functions

### Unified Error Types

- **Create a common error struct** — Build `MyApp.Error` with `:code`, `:message`, `:meta` for consistent error handling
- **Wrap errors with context** — Return `{:error, {:users, :not_found}}` instead of `{:error, :not_found}`

## Pipelines

- **Don't pipe into `case` statements** — Assign intermediate results to variables instead
- **Don't hide higher-order functions** — Write functions that operate on single items, use `Enum.map/2` etc. in calling code
- **Avoid single-step pipelines** — Write `String.downcase(str)` instead of `str |> String.downcase()`
- **Start pipelines with bare variables** — Write `str |> String.trim() |> String.downcase()` not `String.trim(str) |> String.downcase()`
- **Use parentheses in pipelines** — Write `|> String.downcase()` not `|> String.downcase`

## Control Flow

### Choosing the Right Construct

- **Use `if` for boolean conditions** — Don't abuse function clauses for simple true/false branching
- **Use `case` for pattern matching on data** — Avoid unnecessary indirection through private functions
- **Use `cond` for multiple boolean conditions** — Scales better than nested `if` or function clauses with boolean args. **Never use `else if`**
- **Use `with` for chaining fallible operations** — When operations return `{:ok, _} | {:error, _}`
- **Use `for` comprehensions for filter + map + reduce** — Often clearer than chained `Enum` calls
- **Variable Rebinding** — In block expressions (`if`, `case`, `cond`), explicitly rebind variables if the result is needed (e.g., `socket = if ... do ... end`). Do not rebind inside the block expecting it to leak out

### Pattern Matching and Guards

- **State what you expect, not what you don't** — Use `when is_binary(req)` instead of `when not is_nil(req)`
- **Use `and/2`, `or/2`, `not/1` for boolean operands** — Prefer over `&&`, `||`, `!` when both sides are booleans
- **Pattern match in function heads** — Put `%User{} = user` in the signature, not the body
- **Be assertive** — Prefer crashing on unexpected input over defensive coding that hides bugs

## Module and Code Organization

### Architecture (Core/Interface Pattern)

- **Separate core logic from interface** — Core handles business logic; interface handles HTTP, CLI, etc.
- **Interface normalizes input, core expects well-shaped data** — Validate and transform in controllers/handlers
- **Core returns well-shaped results** — Don't leak raw Ecto changesets or HTTP responses

### Module Structure

- **One module per file** — Unless a module is only used internally by another. **Never** nest multiple modules in the same file to avoid cyclic dependencies
- **Keep related code together** — Place private helpers near their public function, not at the bottom
- **Order module contents consistently**: `@moduledoc`, `@behaviour`, `use`, `import`, `require`, `alias`, attributes, `defstruct`, types, callbacks, functions
- **Use `__MODULE__`** — Refer to the current module by its pseudo-variable

### Functions

- **Eliminate single-callsite functions** — If a function is only called once and doesn't improve readability, inline it
- **Group related function clauses** — Run single-line `def`s together; separate multiline `def`s with blank lines
- **Avoid unrelated multi-clause functions** — If clauses do completely different things, split into separate functions
- **Keep the last expression explicit** — Don't hide return values in helper functions

## Types and Data

- **Avoid primitive obsession** — Use structs/maps for complex data, not strings or floats (e.g., for addresses, money)
- **Elixirfy external data** — Convert JSON from APIs to structs at the boundary, not raw maps throughout
- **Add typespecs to public functions** — Documents inputs/outputs clearly, enables Dialyzer
- **Keep structs under 32 fields** — Larger structs lose BEAM optimizations; nest or split if needed
- **Date/Time** — Use the standard library (`Time`, `Date`, `DateTime`, `Calendar`) or `date_time_parser` for parsing. Avoid other deps
- **Security** — Don't use `String.to_atom/1` on user input

## Mix and Tooling

- **Task usage** — Read docs/options (`mix help task_name`)
- **Testing** — Debug with `mix test test/file.exs` or `mix test --failed`
- **Dependencies** — `mix deps.clean --all` is rarely needed; avoid it

## Process Design

- **Don't use processes for code organization** — Processes are for concurrency, shared state, and error isolation
- **Encapsulate process interaction** — Don't scatter `GenServer.call/cast` across modules; wrap in a module API
- **Avoid sending large messages** — Data is copied between processes; send only what's needed
- **Provide child specs, not supervision trees** — Let users add your processes to their own supervisors
- **Child Specs** — Use `{DynamicSupervisor, name: Name}` syntax for named processes in specs
- **Concurrency** — Use `Task.async_stream(collection, callback, timeout: :infinity)` for concurrent enumeration

## Library Design

- **Don't use application config for libraries** — Pass options as function arguments or keyword lists
- **Stay in your namespace** — Library `:my_lib` should only define modules under `MyLib.*`
- **Provide both `foo` and `foo!` variants** — Let callers choose whether to handle errors or raise
- **HTTP Requests** — Use `:req` (`Req`) library. Avoid `:httpoison`, `:tesla`, `:httpc`

## Macros and Metaprogramming

- **Avoid macros when functions suffice** — Macros are harder to understand and debug
- **Don't propagate dependencies from macros** — Avoid `import` inside `__using__/1` that surprises users
- **Keep macro-generated code small** — Large expansions slow compilation and bloat artifacts

## Testing

- **Use `for` comprehensions for collection assertions** — `for post <- posts, do: assert %Post{} = post` gives better failure messages than `assert Enum.all?(...)`
- **Put expected result on the right** — `assert actual == expected` unless pattern matching
- **Test through the interface** — Prefer integration-style tests that call public APIs
- **Generate unique test data** — Enables concurrent test execution

## Formatting and Style

- **Run `mix format`** — Use the official formatter for all code
- **Limit lines to 98 characters** — Configure in `.formatter.exs` if needed
- **Use blank lines to separate logical sections** — Between `def`s, after multiline assignments
- **Avoid trailing whitespace and ensure final newline**
- **Write self-documenting code** — Prefer clear names over comments explaining unclear code

## Naming Conventions

- **`snake_case`** for atoms, variables, functions, and file names
- **`CamelCase`** for modules (keep acronyms uppercase: `HTTPClient`, `XMLParser`)
- **Trailing `?`** for boolean-returning functions: `valid?`, `empty?`. **Do not use `is_` prefix for these**
- **`is_` prefix** — Reserved for guard-safe boolean checks: `is_valid`, `is_empty`
- **`Error` suffix** for exception modules: `BadRequestError`
- **Avoid `do_` prefix** for private functions — Find more descriptive names

## Documentation

- **Always include `@moduledoc`** — Use `@moduledoc false` if intentionally undocumented
- **Place `@doc` and `@spec` immediately before `def`** — No blank line between them
- **Use heredocs with markdown** — For multi-line documentation
- **Document public functions, skip private ones** — Unless complexity warrants it

## Ecto and Database

- **Input Validation** — Use `Ecto.Schema.embedded_schema` for input validation (separate from DB schemas).
    - **Use `@primary_key false`**
    - **Define `changeset/2` and `parse/1`**
    - **Transform to domain structs explicitly**
- **Preloading** — Always preload associations used in templates
- **Schema Fields** — Always use `:string` type (even for text columns)
- **Changesets** — Use `Ecto.Changeset.get_field/2` to access fields. `validate_number` doesn't need `:allow_nil`
- **Security** — Never cast programmatically set fields (like `user_id`); set them explicitly
- **Seeds** — Remember to import `Ecto.Query`

### Example Embedded Schema

```elixir
defmodule MyApp.Requests.CreateOrder do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key false
  embedded_schema do
    field :user_id, :integer
    field :product_sku, :string
    field :quantity, :integer, default: 1
    field :notes, :string
  end

  @required ~w(user_id product_sku)a
  @optional ~w(quantity notes)a

  def changeset(params) do
    %__MODULE__{}
    |> cast(params, @required ++ @optional)
    |> validate_required(@required)
    |> validate_number(:quantity, greater_than: 0)
  end

  def parse(params) do
    case changeset(params) do
      %{valid?: true} = cs -> {:ok, apply_changes(cs)}
      cs -> {:error, cs}
    end
  end
end
```

## Phoenix Framework

### General Guidelines

- **Router** — Be mindful of `scope` aliases. Do not create redundant aliases.
- **Views** — `Phoenix.View` is not needed; do not use it.
- **Project Structure** — Use `mix precommit` (if available).

### Phoenix HTML & HEEx

- **Templates** — Always use `~H` or `.html.heex`.
- **Forms** — Use `Phoenix.Component` functions (`to_form/2`, `<.form>`, `<.input>`). Avoid deprecated `form_for`.
    - **Form Access** — Access forms via `@form[:field]`. Never access changesets directly in templates (`@changeset[:field]`).
    - **IDs** — Add unique DOM IDs to key elements.
- **Interpolation** — Use `{}` for attributes and tag bodies. Use `<%= %>` only for blocks (if, for, case) inside tag bodies.
    - **No Curly** — Use `phx-no-curly-interpolation` for code blocks containing literal curlies.
- **Classes** — Use list syntax `class={["base", @cond && "active"]}` for conditional classes.
- **Loops** — Use `for` comprehensions (`<%= for ... %>`), never `Enum.each`.
- **Comments** — Use `<%!-- --%>`.

### Phoenix LiveView

- **Navigation** — Use `<.link navigate={...}>` or `patch`. Avoid `live_redirect`.
- **Naming** — Suffix LiveViews with `Live` (e.g., `WeatherLive`).
- **Streams** — Use streams for collections to avoid memory issues.
    - **Update** — `phx-update="stream"`.
    - **Filtering** — Refetch and reset (`reset: true`). Streams are not enumerable.
    - **Counting** — Track counts separately; streams don't support counting.
- **Testing** — Use `Phoenix.LiveViewTest` and `LazyHTML`.
    - **Selectors** — Test against DOM IDs, not raw text.
    - **Forms** — Test using `render_submit` and `render_change`.
- **Form Handling** — Create forms from params or changesets using `to_form/2` in the LiveView.

### Authentication

- **Router Organization**:
    - `live_session :current_user` — For routes working with/without auth.
    - `live_session :require_authenticated_user` — For routes requiring auth.
    - **Don't duplicate** live_session names.
- **Access** — Use `current_scope.user` (or assigned user from the session). Do not assume `@current_user` is automagically there without the right mount.

### UI/UX and Assets

- **Tailwind CSS** — Use v4 import syntax in `app.css`. Avoid `@apply`.
- **Components** — Write custom Tailwind components; avoid heavy UI libraries if possible.
- **Icons** — Use `<.icon name="hero-..." />`.
- **JavaScript** — No inline `<script>` tags. Use hooks in `assets/js`.
- **Design** — Focus on polished, responsive, world-class UI with micro-interactions.
