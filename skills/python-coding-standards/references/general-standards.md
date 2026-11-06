# General Standards

These are the shared standards. Rules with a Ruff code are enforced by the configuration in [`../assets/pyproject.ruff-template.toml`](../assets/pyproject.ruff-template.toml); conventions without a rule code are review guidance. The full rule-family map and configuration details are in [`ruff-config.md`](ruff-config.md); version-specific syntax and API rules are in [`version-compatibility.md`](version-compatibility.md). If this summary and the configuration disagree, the configuration wins.

## Module Organization and Imports

- Order module members as: module docstring, imports, `__all__` (only in a re-exporting package `__init__.py`), constants, type aliases, classes, functions, and the `if __name__ == "__main__":` guard. Dependency order overrides this default: define a name before anything that evaluates it at import time.
- Within a module or class, keep public functions and methods before internal `_` helpers; dependency order takes precedence. Dunder methods keep their conventional placement (constructors and protocol methods before public methods).
- Keep module-level code to imports, definitions, constants, and the entry guard. Do not run I/O, network calls, global state mutation, or business logic at import time.
- In a package `__init__.py` that re-exports, list every public name in `__all__` and keep it sorted (`RUF022`). `F401` treats names in `__all__` as used, and `F822` catches names that are not defined. Ordinary modules do not declare `__all__`.
- Prefer absolute imports for clarity and refactoring safety. Relative imports (e.g. `from .models import User`) are acceptable within a package or module tree when aligned with established project practice, but the chosen style must remain consistent across the project. Import order is enforced by isort (`I`), with `typing_extensions` and `_typeshed` grouped with the standard library. Projects requiring strict enforcement of absolute imports can enable `TID252`.
- Keep imports at module top (`E402`, `PLC0415`). A function-level import is allowed only for a circular or optional dependency and needs `# noqa: PLC0415` with the reason. Do not use wildcard imports (`F403`).
- If a module-level class shadows an imported class it subclasses or wraps, prefer `import module` and qualify (`pathlib.Path`); when an alias is necessary, use a private `_Base` prefix (`from pathlib import Path as _BasePath`).
- Use `pathlib` instead of `os.path` for path manipulation (`PTH`), and prefer `Path.open()` over the built-in `open()` (`PTH123`).
- Pass `encoding=` explicitly for text I/O when the supported floor is below 3.15; see [`version-compatibility.md`](version-compatibility.md). `PLW1514` is preview, so this is review guidance.
- For a long module with distinct logical sections, a separator comment with a short noun phrase is acceptable; use it sparingly and keep it within the line limit:

  ```python
  # ===================================================================================
  # Section topic
  # ===================================================================================
  ```

## Naming

PEP 8 covers casing; these defaults cover the semantic choices Ruff cannot check. Follow the project's established names when they differ, and record project-specific deviations in the project's own standards documentation.

### Vocabulary and Consistency
- Use one name for one concept and one verb for one action in the project's vocabulary, especially in public APIs (`sync_fetch`/`async_fetch`). Do not mix `user`/`account`, `fetch`/`load`, or `config`/`cfg`.
- Name exception variables with one of `e`, `err`, or `exc`, and use the same form throughout the project; do not mix them. An established project convention wins.
- Prefer full words in public names. Common acronyms (`id`, `url`, `api`, `http`) are acceptable; write acronyms in lowercase in snake_case names (`http_client`). Avoid standalone `id` shadowing (`A`).
- Modules and packages used only internally use a leading underscore (`_helpers.py`, `_internal/`). Avoid `utils`, `helpers`, and `common` as dumping grounds.
- Internal functions, methods, classes, and module-level variables use a leading underscore (`_load_config`). Do not underscore local variables for privacy; use `_` only for intentionally unused values.
- Do not use `__name` for privacy; double underscores trigger name mangling and are only for avoiding subclass attribute collisions. Do not invent new dunder names.

### Variables, Constants, and Mappings
- Use plural nouns for collections (`users`) and `<value>_by_<key>` for mappings (`users_by_id`). Do not add `_list` or `_dict`.
- Name boolean variables, attributes, and properties with positive predicates (`is_running`, `has_access`, `enabled`). Avoid `_flag` and negative names (`is_not_ready`, `no_cache`).
- Use `UPPER_SNAKE_CASE` for module/class constants (`MAX_RETRIES`) and enum members (`Color.RED`).
- Include units for numeric values (`timeout_seconds`, `size_bytes`, `retry_delay_ms`).
- For iteration, unpacking, and local bindings, follow a three-tier semantic hierarchy:
  - **Domain semantics (preferred)**: Use singular nouns for sequences (`for user in users`), and concrete entity roles when unpacking mappings, tuples, or enumerations (`for user_id, profile in users_by_id.items()`, `for line_no, line in enumerate(lines)`, `for user, role in zip(users, roles)`, `lambda user: user.name`).
  - **Structural fallback**: Fall back to full role nouns (`key, value`, `item`, `index`) only when operating on abstract or generic containers lacking domain entities.
  - **Compact shorthand**: Restrict single-letter abbreviations (`k, v`, `i, j`, `x, y`) strictly to obvious micro-scopes, such as single-line comprehensions (`{k: str(v) for k, v in data.items()}`), pure arithmetic lambdas (`lambda x, y: x + y`), or simple counting indices. Do not use single-letter variables in multi-line business logic.

  ```python
  # Bad: single-letter or generic names in multi-line business logic
  for k, v in users_by_id.items():
      send_notification(k, v.email)

  # Good: domain-specific semantic unpacking (preferred)
  for user_id, profile in users_by_id.items():
      send_notification(user_id, profile.email)

  # Good: compact shorthand strictly in a single-line micro-scope
  user_names = {uid: u.name for uid, u in users_by_id.items()}
  ```

### Functions, Methods, and Converters
- Use verb phrases (`load_config`). Avoid bare `handle`/`process` without an object (`process_payment`).
- Alternate constructors use `from_*`; other factories use `create_*`, `build_*`, or `make_*`.
- Properties use a noun or adjective (`size`, `is_empty`). Conversion helpers use `to_*` (new value) or `as_*` (view/cast).
- Functions returning iterators or generators use `iter_*` or `iterate_*` for generic iteration; prefer `iter_*` when the project has no convention, and use a specific semantic name (`stream_*`, etc.) when one applies.
- Keep one generic form across the project, except when following a standard library or third-party API's naming.
- Do not add `async_` to async functions by default. When providing paired sync/async functions or methods, prefix both with `sync_` and `async_` (`sync_fetch`/`async_fetch`, `sync_iter_users`/`async_iter_users`).

### Classes, Mixins, and Types
- Classes use singular nouns or noun phrases (`User`, `ConnectionPool`). Protocols may use adjectives (`Iterable`). Paired async classes use `Async` (`AsyncClient`).
- Write a single abbreviation in uppercase (`HTTPClient`); treat adjacent abbreviations as words (`HttpUrl`).
- End mixin classes with `Mixin` (`LoggingMixin`).
- Error exceptions end with `Error` (`ConfigError`, enforced by `N818`); warning classes end with `Warning`; control-flow signals use semantic names (`StopIteration`).
- Type variables use `T`, `U`, `V` or descriptive names (`UserT`); mappings use `KT`/`VT`; parameter specs use `P`; variadic tuples use `Ts`.

## Type Annotations

- Annotate every function parameter and return value (`ANN`); `self` and `cls` are exempt. Annotate class attributes and empty-container initializations (`items: list[str] = []`).
- Use built-in generics (`list[str]`, `dict[str, int]`) and PEP 604 unions (`X | Y`); do not use `Optional`, `Union`, `typing.List`, or `typing.Dict` (`UP006`, `UP007`, `UP035`, `UP045`). Always parameterize containers and generics (`Callable[[int], str]`).
- On Python 3.12+, use PEP 695 generics and type aliases (`class Box[T]: ...`, `def first[T](...): ...`, `type IntList = list[int]`) (`UP040`, `UP046`, `UP047`). On 3.11, use `TypeVar`, `Generic`, and `TypeAlias`; see [`version-compatibility.md`](version-compatibility.md).
- Prefer precise types over `Any`: use `object` when any value is acceptable, or a `Protocol` when behavior is expected. `ANN401` rejects `Any` in function signatures.
- Use `ClassVar[T]` for mutable class-level state (`RUF012`) and non-field attributes in dataclasses. Use `Final[T]` for constants that must not be rebound.
- A method returning its own instance annotates `-> Self`, including fluent setters and `__enter__`.
- Move typing-only imports behind `if TYPE_CHECKING:` (`TC001`–`TC003`), along with `@overload` signatures not needed at runtime.
- Do not use `from __future__ import annotations`; `TID251` bans it. Quote forward references that are evaluated before definition, including types imported under `TYPE_CHECKING`.
- Keep normal runtime imports for base classes, `isinstance` checks, decorators (`TC004`), and frameworks that resolve annotations at runtime.

## Functions and Data

### Function Design
- Keep each function at one level of abstraction; extract helpers instead of deep nesting, and use guard clauses/early returns (`RET505`–`RET508`).
- Use a single blank line inside a function or method body only when the body is long enough to have distinct phases (roughly 20 lines or more), such as validation, the main computation, and result assembly. Keep short functions compact; do not insert blank lines between individual statements. If a function needs several such separators, extract a helper instead; `ruff format` collapses consecutive blank lines inside a body to one.

  ```python
  # Good: a long body uses single blank lines between major phases
  def aggregate_metrics(events: list[Event], *, window_seconds: int) -> Metrics:
      if window_seconds <= 0:
          raise ValueError("window_seconds must be positive")

      buckets: dict[int, list[Event]] = {}
      for event in events:
          bucket = int(event.timestamp) // window_seconds
          buckets.setdefault(bucket, []).append(event)

      counts = {bucket: len(items) for bucket, items in buckets.items()}
      durations = {
          bucket: sum(event.duration_ms for event in items)
          for bucket, items in buckets.items()
      }
      peak = max(counts.values(), default=0)
      total = sum(durations.values())

      return Metrics(
          counts=counts,
          durations=durations,
          peak=peak,
          total=total,
      )
  ```

- Make boolean parameters keyword-only with `*`; do the same when several parameters are optional (`FBT001`, `FBT002`). Do not hide a public API signature behind `**kwargs` (decorators and forwarding are exempt). When parameters form a cohesive group or no longer fit on one line, introduce a dataclass, `TypedDict`, or `NamedTuple` parameter object.

  ```python
  # Bad: positional boolean arguments obscure meaning at call site (FBT001)
  def render(text: str, cache: bool = True, strip: bool = False) -> str: ...
  render("hello", True, False)

  # Good: keyword-only boolean parameters (*) ensure call-site clarity
  def render(text: str, *, cache: bool = True, strip: bool = False) -> str: ...
  render("hello", cache=True, strip=False)
  ```

- Never use a mutable object as a default value; use `field(default_factory=...)` or a `None` sentinel created inside the function (`B006`, `RUF012`).
- Do not return results through mutable arguments, pass state through `global`, or perform I/O in the middle of a computation. Use `nonlocal` only for closure-local state, and keep input/output at the boundary (`PLW0603`).

### Async
- Do not run blocking I/O or CPU-bound work inside an `async def`; offload it with `asyncio.to_thread` or an executor (`ASYNC`).
- Use `asyncio.TaskGroup` (3.11+) for structured concurrency, and bound waits with `asyncio.timeout`.
- Do not swallow `CancelledError`; let cancellation propagate.

### Comprehensions
- Keep a comprehension or generator expression to one `for` and at most one `if`; use a loop for nested comprehensions, multiple filters, or multi-step logic.
- A second `for` is allowed only to flatten a 2D structure (`[x for row in matrix for x in row]`); a parallel double loop uses `itertools.product` (`[f(x, y) for x, y in product(xs, ys)]`); a dependent nested loop uses an explicit loop.
- `itertools.product` materializes its inputs; for large or infinite iterables where streaming matters, keep the nested generator and state why.
- `C4` covers mechanical simplifications; the structural limits above are review guidance.

```python
# Bad: nested dependent loops and multiple filters in a comprehension
users = [u.name for group in groups if group.is_active for u in group.users if u.is_admin]

# Good: flatten a 2D structure
flat_matrix = [cell for row in matrix for cell in row]

# Good: parallel cartesian product
coords = [f(x, y) for x, y in itertools.product(xs, ys)]

# Good: explicit loop for multi-step or dependent logic
users: list[str] = []
for group in groups:
    if not group.is_active:
        continue
    users.extend(u.name for u in group.users if u.is_admin)
```

### Data Modeling
- Validate external input once at the boundary with the project's validation mechanism (for example, pydantic), then pass typed data internally; do not re-validate the same input in inner functions. Use a dataclass, `TypedDict`, or `NamedTuple` for structured internal data, not a bare `dict` or `tuple`.
- Model immutable value objects with `@dataclass(frozen=True, slots=True)`; use `tuple`/`frozenset` for sequence and set fields, and omit `slots=True` when the object needs `__dict__`, weak references, or a specific pickling/ORM contract.
- For a value object's mapping field, use the built-in `frozendict` on Python 3.15+, or a maintained immutable mapping implementation (third-party or custom) on earlier versions; record the choice in the project's own standards documentation. `MappingProxyType` is a read-only view, not a `frozendict` equivalent: it is unhashable and reflects mutations of the wrapped mapping, so it does not satisfy deep immutability.
- Use timezone-aware datetimes: `datetime.now(UTC)` or `datetime.now(timezone.utc)`, never `utcnow()` or a naive `now()`. Parse and store in UTC, convert at the edges (`DTZ`).
- Validate invariants in `__post_init__` or a validator; keep constructors free of I/O and expensive recomputation.
- Use `Enum` for a closed set of named values. Replace a class that only holds static methods with module-level functions, unless a framework requires the class.

## Docstrings and Comments

### Docstrings

- Use the Google docstring convention (`D`) for public modules, classes, methods, and functions; internal APIs may use a concise summary. Package docstrings (`D104`), magic methods (`D105`), and `__init__` (`D107`) are exempt.
- Structure docstrings with a one-line summary, a blank line, and sections in order: `Args`, `Returns`, `Yields`, `Raises`, `Attributes`.
- In `Args`, list parameters in signature order using `name (type[, optional]):`; state defaults in the description. In `Attributes`, use `name (type):` and omit the type when the annotation is adjacent.
- For `Returns`, `Yields`, and `Raises`, the type is the first line with the description indented below. Use `Yields` for generators; omit `Returns` when returning `None`.
- For `@dataclass`, `TypedDict`, and `NamedTuple`, prefer a per-field PEP 258 docstring where the project's tooling reads it (typically `@dataclass`); an `Attributes:` section is also acceptable. Keep one style per project.
- Write docstrings in either imperative mood (PEP 257 default: "Fetch users...") or declarative sentences ("Fetches users..."), keeping the choice consistent across the project; end sentences with punctuation in the document's language and keep section headers in English.

```python
def fetch_users(query: str, *, limit: int = 100) -> list[User]:
    """Fetch matching users from the directory.

    Args:
        query (str):
            Search term matching name or email address.
        limit (int, optional):
            Maximum records to retrieve. Defaults to 100.

    Returns:
        list[User]:
            Matching user records sorted in ascending creation order.

    Raises:
        ConnectionTimeoutError:
            If the remote directory service fails to respond.
    """
```

```python
# Good: document each data-class field next to its definition
@dataclass(frozen=True, slots=True)
class RetryPolicy:
    """Configure retry behavior for outbound requests."""

    max_attempts: int = 3
    """Maximum number of attempts, including the first."""
    base_delay_seconds: float = 0.5
    """Initial delay before the first retry."""
    max_delay_seconds: float = 30.0
    """Upper bound for the exponential backoff delay."""
```

### Comments

- Use comments only for non-obvious constraints, trade-offs, upstream defects, and reference links. Do not narrate what the code does or restate annotations.
- Delete commented-out code (`ERA`), stale comments, and ownerless `TODO`/`FIXME` markers.

## Exception Handling

- Catch specific exceptions; bare `except:` is forbidden (`E722`). `except Exception`/`BaseException` is permitted only at process boundaries (CLI, server middleware, worker loop) and must log the traceback (`TRY400`) or re-raise (`BLE001`).
- Never swallow exceptions silently (`S110`, `S112`). Translate with `raise NewError(...) from exc` (`B904`).
- For intentional, safe-to-ignore exceptions, use `with contextlib.suppress(SpecificError):` containing only the raising statement. Never suppress `Exception` or `BaseException`.
- Keep `try` blocks minimal: place only the raising call inside `try`; move validations and early raises before it (`TRY301`), and returns to `else:` (`TRY300`). Manage resources with `with`/`async with`.

  ```python
  # Bad: giant try block, broad catch, and lost traceback
  try:
      raw = fetch_payload(url)
      parsed = json.loads(raw)
      return save_to_db(parsed)
  except Exception:
      raise DatabaseError("Save failed")

  # Good: minimal try, specific catch, else return, and chained exception
  raw = fetch_payload(url)
  try:
      parsed = json.loads(raw)
  except json.JSONDecodeError as exc:
      raise PayloadFormatError(f"Malformed json from {url}") from exc
  else:
      return save_to_db(parsed)
  ```

- Raise project-specific exception types for errors callers should handle; do not leak raw `KeyError` or `AttributeError`. Keep secrets and tokens out of exception messages and logs.
- Do not use `assert` for input validation (`S101`); raise `ValueError` or `TypeError`. Use `assert` only for genuine internal invariants with `# noqa: S101` and a reason.

## Testing

- Name tests `test_<behavior>_<condition>_<expected>`; describe the behavior, not a sequence number (`test_1`).
- Organize each test as arrange, act, assert; one test verifies one behavior. When the same assertion repeats across cases, use the framework's parameterization (`pytest.mark.parametrize` for pytest, `subTest` for unittest) instead of a loop with branches.
- Use the framework's native assertion style: plain `assert` in pytest, `assertEqual`/`assertTrue`/etc. in unittest. Assert exception types with `pytest.raises` or `unittest.assertRaises`/`assertRaisesRegex`, and match the message when it is part of the contract.
- Keep unit tests isolated: no network, real clock, unseeded randomness, or execution order. Inject or freeze external dependencies (clients, clocks, RNG, filesystem, environment); each test must pass alone and in any order.
- When fixing a defect, add a test that reproduces it; confirm it fails before the fix when practical.

## Security, Logging, and Suppressions

- Use the project's logging facility (e.g., standard `logging`, loguru, structlog) for diagnostics; do not use `print` or `pprint` (`T20`).
- Use appropriate levels and the facility's lazy formatting (`logger.info("x=%s", x)` for stdlib `logging`, `logger.info("x={}", x)` for loguru); do not build messages with f-strings or concatenation.
- Log exceptions with the facility's `exception()` method inside handlers (`TRY400`), and keep secrets and personal data out of logs.
- Never execute `eval` or `exec` on untrusted input (`S102`, `S307`). Prefer structured parsers, `ast.literal_eval`, or dispatch tables.
- Never hardcode tokens, passwords, API keys, or personal data in code, tests, examples, or docs (`S105`–`S107`). Read credentials from the environment or secret managers.
- Do not use `shell=True` with untrusted input (`S602`); pass an argument list instead.
- Do not deserialize untrusted data with `pickle` or `marshal` (`S301`, `S302`); use JSON or a typed parser.
- Do not use bare `# noqa` or `# type: ignore`. Always include the specific rule code and a reason (`PGH003`, `PGH004`): `# noqa: E501  # reason`, `# type: ignore[arg-type]  # reason`.
- Prefer fixing the type, introducing a `Protocol`, or adjusting the shared configuration with a documented reason over adding inline suppressions.
