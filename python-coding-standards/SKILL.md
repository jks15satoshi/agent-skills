---
name: python-coding-standards
description: Lint, format, and enforce modern Python coding standards using Ruff, and review naming, typing, structure, testing, docstring, and exception-handling conventions. Use this skill when the user wants to check, format, or refactor Python code, review naming, typing, structure, testing, docstring, or exception-handling conventions, configure Ruff in pyproject.toml, or ensure syntax and typing compatibility for Python 3.11+.
compatibility: Requires Python 3.11+ and Ruff. uv is recommended.
license: MIT
metadata:
  author: jks15satoshi
  version: "1.0.0"
---

# Python Coding Standards

## Overview

The shared Ruff rule set and the project's declared minimum Python version are the source of truth for enforceable rules. This skill routes code standards, structure, testing, version, and configuration work to the reference that owns each rule; prose guidance covers only conventions Ruff cannot check and never overrides the project's established conventions.

## Workflow

- [ ] Step 1: Determine the minimum supported Python version. Read [`references/python-versions.md`](references/python-versions.md) for the detection order. If no floor is declared, ask; do not guess.
- [ ] Step 2: Merge [`assets/pyproject.ruff-template.toml`](assets/pyproject.ruff-template.toml) into the root `pyproject.toml` without overwriting existing project configuration. Read [`references/ruff-config.md`](references/ruff-config.md) before changing existing Ruff settings.
- [ ] Step 3: Write code that runs on the minimum version and follows the general standards. Read [`references/general-standards.md`](references/general-standards.md); for syntax and standard-library availability, read [`references/python-versions.md`](references/python-versions.md). If the task touches a project-specific library, framework, or internal API, read [`references/project-specific-standards.md`](references/project-specific-standards.md).
- [ ] Step 4: Run Ruff and fix violations — see the commands below.
- [ ] Step 5: Verify the supported version range. Read [`references/python-versions.md`](references/python-versions.md) for the test matrix.

### Run Ruff

Choose the runner once:

- **uv project, Ruff installed as a dependency** (`uv.lock` or `[tool.uv]` present): `uv run ruff`. This uses the Ruff version pinned by the project.
- **uv project, Ruff not in the project dependencies**: `uvx ruff`, or `uvx --from ruff==<version> ruff` to pin it. Add Ruff with `uv add --dev ruff` if the project should own the version.
- **No uv**: `ruff`.

The commands below use `uv run ruff`. Drop `uv run` for a direct `ruff` invocation, or replace `uv run ruff` with `uvx ruff` when Ruff is not installed in the project.

```bash
# Focused change (default)
uv run ruff check --fix <path/to/touched_file.py>
uv run ruff format <path/to/touched_file.py>

# Whole-project run (only for greenfield setup or explicit full-repo cleanup)
uv run ruff check --fix .
uv run ruff format .
uv run ruff check .
uv run ruff format --check .
```

Fix violations in the code rather than suppressing them. When a suppression is unavoidable, use a rule code and a reason. Review fixes before accepting them; never use `--unsafe-fixes` without manual inspection.

## Gotchas

- A copied `target-version` can silently target the wrong Python version. Prefer `requires-python`; if both exist, `target-version` wins.
- `target-version` does not prove that a standard-library API exists at the floor.
- Newer syntax cannot be made compatible with an older interpreter by wrapping it in `sys.version_info`; the parser reads the whole file.
- `from __future__ import annotations` is banned by `TID251`. Use quoted annotations for forward references and for types imported under `TYPE_CHECKING`.
- The `tests/**` per-file ignores apply only to tests; do not extend them to production code.
- Do not run `ruff check --fix .` across an existing repository unless explicitly requested; use focused file arguments.
- Never use `--unsafe-fixes` without manually inspecting the diff.
- Do not enable lint rules that conflict with `ruff format` (e.g. `COM812`, `ISC001`).
- `N818` flags every custom `Exception` subclass that lacks the `Error` suffix, including non-error control-flow exceptions; `BaseException` and `Warning` subclasses are not checked.
- `S110`/`S112` check typed exceptions only because the template sets `check-typed-exception = true`; `contextlib.suppress` is the sanctioned intentional ignore.
- `BLE001` accepts `except Exception` when the handler logs the exception with its traceback (`exception()` or `exc_info=True`) or re-raises; a handler that only returns a failure needs `# noqa: BLE001` with a reason, and a logged handler must not add it (`RUF100`).
- `TRY003` is ignored on purpose: it rejects multi-word messages at the raise site. Enable it in a project that wants class-owned messages everywhere.
- `UP040`/`UP046`/`UP047` only fire at a 3.12+ target; `ANN` covers function signatures only and `RUF012` covers mutable class defaults only, so `ClassVar`, `Final`, `Self`, and bare generics remain review guidance.
- `TC004` catches direct runtime uses (`isinstance`, base classes, decorators), but not `get_type_hints`, `get_overloads`, or framework annotation resolution; keep those imports at runtime manually.
- `PGH003`/`PGH004` require codes on `type: ignore`/`noqa`; the reason is review guidance.
- `D415` is ignored because it accepts only ASCII `.`, `?`, and `!`; terminal punctuation in non-English docstrings is review guidance.
- `D417` checks only docstrings that contain an `Args` section; a public function without one can still be under-documented.
- Attribute docstrings on data-class fields are not checked by Ruff; `Attributes:` is the portable default, and below-field docs are only an option for `@dataclass` when the project's tooling reads PEP 258 attribute docstrings.
- `PLC0415` enforces top-level imports; a function-level import needs `# noqa: PLC0415` with the circular/optional reason. Absolute imports are recommended, while intra-package relative imports are acceptable if consistent across the project; `TID252` is available for strict enforcement. `RUF022` sorts `__all__`, and `F401`/`F822` check re-exports in `__init__.py`.
- `T20` bans `print`/`pprint`, and `S102`/`S307` ban every `eval`/`exec`; deliberate CLI output or trusted evaluation needs a coded `# noqa` with a reason. The logging facility can be stdlib `logging`, loguru, or structlog.
- `S105`–`S107` catch common password patterns but not every token/key or personal data; the `tests/**` ignores for `S105`/`S106` are only for obvious dummy values.
- `C4` covers mechanical simplifications; the single-level, flattening-only double-loop, and `itertools.product` limits are review guidance.
- `B006`/`RUF012` enforce mutable defaults, `RET505`–`RET508` early-return mechanics, `FBT001`/`FBT002` keyword-only boolean definitions, and `PLW0603` `global`; the rest of Functions and Data is review guidance. `MappingProxyType` is a read-only view, not a `frozendict`: it is unhashable and reflects mutations; below Python 3.15 use a maintained immutable mapping.
- `DTZ` enforces timezone-aware datetime construction; `PLW1514` is preview and is not enabled, so explicit `encoding=` is review guidance.
- `PT` covers pytest style; test naming, arrange-act-assert, and isolation are review guidance. `S101` is ignored in tests so `assert` is allowed, and `PT009`/`PT027` are ignored globally so unittest-style `assertEqual`/`assertRaises` remain valid; a pytest project can re-enable them.
- Comments and docstrings may use any language the project chooses.

## Validation

Before finishing:

- [ ] `requires-python` is declared and matches the version used for compatibility decisions.
- [ ] Ruff configuration lives in the root `pyproject.toml`.
- [ ] `ruff check` and `ruff format --check` pass on the affected files, using the runner chosen in Step 4.
- [ ] Public names follow the Naming conventions in [`general-standards.md`](references/general-standards.md); one concept has one name and one action has one verb.
- [ ] Exception handling follows [`general-standards.md`](references/general-standards.md): specific catches, no silent swallow, explicit ignores, no secrets in messages.
- [ ] Typing follows [`general-standards.md`](references/general-standards.md): annotated signatures and class attributes, parameterized containers, PEP 695 only at 3.12+, typing-only imports/overloads behind `TYPE_CHECKING`, runtime consumers imported normally, coded suppressions.
- [ ] Functions and data follow [`general-standards.md`](references/general-standards.md): one level of abstraction, keyword-only booleans, no mutable defaults, no `global`, boundary validation, frozen value objects, `Enum` for closed sets.
- [ ] Docstrings follow [`general-standards.md`](references/general-standards.md): Google sections, Args order/defaults, `Attributes` or the optional `@dataclass` field docs, terminal punctuation.
- [ ] Module organization follows [`general-standards.md`](references/general-standards.md): member order, public-before-private, consistent top-level imports (absolute preferred), `__all__` only in a re-exporting `__init__.py`.
- [ ] Logging and security follow [`general-standards.md`](references/general-standards.md): a logging facility rather than `print`, no `eval`/`exec` on untrusted input, no real secrets or personal data in code, tests, examples, or docs.
- [ ] Tests follow [`general-standards.md`](references/general-standards.md): behavior names, arrange-act-assert, one behavior, parameterized cases, specific exception assertions, isolation, regression test.
- [ ] Async, date/time, and text I/O follow [`general-standards.md`](references/general-standards.md): no blocking calls in async, timezone-aware datetimes, explicit `encoding=` below 3.15.
- [ ] Tests pass on the minimum and maximum supported versions.
- [ ] Any suppression has a rule code and a reason.

## References

| File | Contents | Read when |
| --- | --- | --- |
| [`general-standards.md`](references/general-standards.md) | module organization, comprehensions, naming, type annotations, functions and data, docstrings, exception handling, testing, security and logging | writing or reviewing code, or naming new APIs |
| [`python-versions.md`](references/python-versions.md) | support matrix, baseline detection, compatibility patterns, end of life | choosing syntax or APIs, or setting the floor |
| [`ruff-config.md`](references/ruff-config.md) | rule set map, target version, per-file ignores, template adoption | configuring or troubleshooting Ruff |
| [`project-specific-standards.md`](references/project-specific-standards.md) | placeholder for project-specific conventions; add the project's rules here | a task touches a project-specific library, framework, or internal API |
| [`pyproject.ruff-template.toml`](assets/pyproject.ruff-template.toml) | canonical Ruff rule set | merging or reviewing configuration |
