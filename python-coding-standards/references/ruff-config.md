# Ruff Configuration

The conventions these rules enforce are summarized in [`general-standards.md`](general-standards.md). The canonical rule set is [`../assets/pyproject.ruff-template.toml`](../assets/pyproject.ruff-template.toml). Merge its `[tool.ruff*]` sections into the project's root `pyproject.toml`.

## Source of truth

- Keep one Ruff configuration per repository.
- Put the configuration in the root `pyproject.toml`.
- Do not add a separate `ruff.toml` or `.ruff.toml` when `pyproject.toml` already holds the rules.
- Do not override `select` or `ignore` in CI or editor commands. The committed configuration is the source of truth.
- Preserve existing project-specific settings when merging the template.

## Target version

- Prefer `[project] requires-python`; Ruff infers the target from it.
- If the project has no PEP 621 metadata, set `target-version` to the minimum supported version.
- If both `requires-python` and `target-version` exist, `target-version` wins. Keep them consistent.
- Do not copy a `target-version` from another workspace without checking the project's floor.

## Template choices

The template intentionally omits one project-specific setting:

- `src`: Ruff defaults to `[".", "src"]`, which covers flat and `src` layouts. Add `src` only for a non-standard layout.

## Rule set map

| Family | Area |
| --- | --- |
| `A` | flake8-builtins: names that shadow builtins |
| `ANN` | flake8-annotations: function annotations |
| `ARG` | flake8-unused-arguments: unused arguments |
| `ASYNC` | flake8-async: blocking calls and async correctness |
| `B` | flake8-bugbear: common bugs and dubious constructs |
| `BLE` | flake8-blind-except: blind `except` clauses |
| `C4` | flake8-comprehensions: comprehension style |
| `C90` | mccabe: cyclomatic complexity |
| `D` | pydocstyle: docstring structure |
| `DTZ` | flake8-datetimez: timezone-aware datetimes |
| `E`, `W` | pycodestyle: errors and warnings |
| `ERA` | eradicate: commented-out code |
| `F` | pyflakes: unused imports, undefined names, and related errors |
| `FBT` | flake8-boolean-trap: boolean positional arguments |
| `FURB` | refurb: modernizations and simplifications |
| `I` | isort: import ordering |
| `PERF` | perflint: performance anti-patterns |
| `PLC0415` | Pylint: imports outside the top-level scope |
| `PLW0603` | Pylint: global statement |
| `PT` | flake8-pytest-style: pytest style |
| `PTH` | flake8-use-pathlib: path handling |
| `RET` | flake8-return: return style |
| `RUF` | Ruff-specific rules |
| `S` | flake8-bandit: security checks |
| `T20` | flake8-print: print statements |
| `TC` | flake8-type-checking: imports used only for typing |
| `TID251` | flake8-tidy-imports: banned APIs |
| `TRY` | tryceratops: exception-handling style |
| `UP` | pyupgrade: modern Python syntax |

`N818` is enabled individually; it is covered under [Naming](#naming). `PGH003` and `PGH004` are enabled individually; they are covered under [Typing](#typing). `PLC0415` is enabled individually; it is covered under [Import ordering](#import-ordering). `FBT001`, `FBT002`, and `PLW0603` are enabled individually; `FBT003` is not.

The configured `ignore` list removes `D104`, `D105`, `D107`, `D415`, `PT009`, `PT027`, `RUF001`, `RUF002`, `RUF003`, and `TRY003`. Comments and docstrings are not restricted to a particular language. `TRY003` is ignored because it rejects multi-word messages at the raise site; see [Exception handling](#exception-handling). `PT009`/`PT027` are ignored globally so unittest-style assertions remain valid; pytest projects can re-enable them.

`C4` covers mechanical comprehension simplifications; the structural limits in [`general-standards.md`](general-standards.md#comprehensions) are review guidance. `PERF401` suggestions are single-level and fit those limits.

`B006` and `RUF012` enforce mutable defaults; `RET505`–`RET508` enforce early-return mechanics; `FBT001`/`FBT002` enforce keyword-only boolean definitions; `PLW0603` flags `global`. The remaining Functions and Data rules are review guidance.

`PT` covers pytest style; `PT009`/`PT027` are ignored globally so unittest-style `assertEqual`/`assertRaises` remain valid. Test naming, arrange-act-assert, and isolation are review guidance.

`DTZ` enforces timezone-aware datetime construction. `PLW1514` is preview and is not enabled, so explicit `encoding=` is review guidance.

## Per-file ignores

The template keeps exceptions local to tests:

- `tests/**`: `ANN`, `ARG`, `D`, `S101`, `S105`, `S106`, `S108`, `S311`.
- `conftest.py`: `ANN`, `ARG`, `D`, `S101`.

The `S105`/`S106` test ignores are for obvious dummy passwords only; they never permit real credentials or personal data.

`S101` is ignored in tests so both pytest `assert` and unittest assertion methods are allowed.

Do not extend these ignores to production code. Add a new ignore only with a comment explaining why the rule is inappropriate for those files.

## Banned API

The template bans `__future__.annotations` through `TID251`:

```toml
[tool.ruff.lint.flake8-tidy-imports.banned-api]
"__future__.annotations".msg = "Do not use `from __future__ import annotations`; use quoted annotations or target-native syntax."
```

Use quoted forward references instead. The related settings are intentional:

```toml
[tool.ruff.lint]
future-annotations = false

[tool.ruff.lint.flake8-type-checking]
quote-annotations = true
```

## Import ordering

`typing_extensions` and `_typeshed` are treated as standard-library imports:

```toml
[tool.ruff.lint.isort]
extra-standard-library = ["_typeshed", "typing_extensions"]
```

Keep that grouping; do not move compatibility imports into the third-party block.

Absolute imports are recommended for clarity and refactoring safety, but relative imports within a package are permitted when aligned with established project practice, provided the style remains consistent across the project. `TID252` is not enabled in the default template. Projects requiring strict enforcement of absolute imports can enable `TID252` and configure:

```toml
[tool.ruff.lint.flake8-tidy-imports]
ban-relative-imports = "all"  # or "parents"
```

`PLC0415` flags imports outside the top level; a function-level import for a circular or optional dependency needs `# noqa: PLC0415` with a reason. `E402` keeps module-level imports before other code.

## Naming

`N818` is enabled individually; the rest of the `N` family is not. `N806` would conflict with local `UPPER_SNAKE_CASE` constants. `N818` requires custom `Exception` subclasses to end with `Error`; it does not check `BaseException` or `Warning` subclasses. A non-error control-flow exception may suppress `N818` with a reason. The semantic naming conventions are review guidance in [`general-standards.md`](general-standards.md); they are not enforced by Ruff.

## Typing

- `ANN` checks function signatures only; `ANN101` and `ANN102` are removed, so `self` and `cls` need no annotation. Class attribute and empty-container annotations are review guidance.
- `ANN401` rejects `Any` in function signatures; `Any` in variables and class attributes is review guidance.
- `RUF012` requires `ClassVar` for mutable class defaults. `ClassVar` elsewhere, `Final`, and `Self` are review guidance.
- `UP006`, `UP007`, `UP035`, and `UP045` enforce built-in generics, PEP 604 unions, and `collections.abc` imports. `UP040`, `UP046`, and `UP047` enforce PEP 695 only at a 3.12+ target; a 3.11 target cannot use PEP 695 syntax.
- `TID251` bans `from __future__ import annotations` (see [Banned API](#banned-api)). `TC001`–`TC003` move imports into `TYPE_CHECKING`; `TC004` moves them out when they are used at runtime. `TC` does not check `@overload` placement. `quote-annotations = true` quotes imports moved by `TC`; quote manually when moving an import or overload yourself.
- `PGH003` and `PGH004` require codes on `type: ignore` and `noqa`; reasons are review guidance.

## Docstrings

The template uses the Google convention:

```toml
[tool.ruff.lint.pydocstyle]
convention = "google"
```

`D104`, `D105`, `D107`, and `D415` are ignored globally. `D415` is ignored because it accepts only ASCII `.`, `?`, and `!`; non-English punctuation such as `。` or `？` would fail. Terminal punctuation is review guidance. `D400` and `D401` are disabled by the Google convention, so declarative first lines are allowed. `D417` checks completeness only when a docstring contains an `Args` section, and it recognizes the English header; keep section headers in English. Attribute docstrings are not checked by Ruff; `Attributes:` is the portable default. Docstring content language is determined by the project, not by this configuration.

## Exception handling

- `S110` and `S112` check typed exceptions because the template sets `check-typed-exception = true`. An intentional ignore uses `contextlib.suppress(SpecificError)`, not `except ...: pass` or `continue`.
- `BLE001` allows `except Exception` when the handler logs the exception with its traceback (`exception()` or `exc_info=True`) or re-raises. A boundary handler that only returns a failure result needs `# noqa: BLE001` with a reason; a logged handler must not add it, because `RUF100` flags an unused `noqa`.
- `TRY003` is ignored because it rejects multi-word messages at the raise site; `TRY300` moves returns to `else`, and `TRY301` moves raises that the same `try` catches out of the `try`.
- `S101` flags `assert` in production code; tests and `conftest.py` ignore it. A genuine internal invariant needs `# noqa: S101` with a reason.

## Formatter

`ruff format` is the formatter. The only configured formatting option is:

```toml
[tool.ruff.format]
docstring-code-format = true
```

Do not run Black, isort, or another formatter alongside Ruff on the same files.

## Adoption

For a uv project, prefix the Ruff commands below with `uv run`; if Ruff is not a project dependency, use `uvx ruff` instead.

1. Merge the template into the root `pyproject.toml`:
   - Inspect existing `[tool.ruff*]` sections. If none exist, append the template sections.
   - If existing Ruff configuration exists, merge tables without duplicating keys or breaking other tool tables (e.g., `[tool.poetry]`, `[tool.pytest]`).
   - Run `ruff check` immediately to validate that the TOML syntax parses correctly.
2. Confirm `requires-python` is present; otherwise set `target-version` explicitly.
3. Run `ruff check --fix` and `ruff format` on the files being changed.
4. Review the diff, especially for unsafe fixes. Never use `--unsafe-fixes` automatically.
5. Run `ruff check` and `ruff format --check` to verify.
6. If a rule is unsuitable, change the shared configuration with a reason instead of scattering `# noqa` comments.

For a focused change, do not reformat unrelated files. A whole-repository cleanup should be a separate, explicit task.
