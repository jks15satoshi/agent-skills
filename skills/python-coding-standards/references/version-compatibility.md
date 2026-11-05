# Python Version Compatibility

This file owns the version policy. General code conventions are in [`general-standards.md`](general-standards.md); Ruff configuration is in [`ruff-config.md`](ruff-config.md).

The feature matrix covers every Python version that still receives maintenance support; a project's declared floor may be higher.

Verified on 2026-11-04 against the [Python Developer's Guide status page](https://devguide.python.org/versions/) and the CPython "What's New" documentation.

## Currently supported versions

| Version | Status | First release | End of life |
| --- | --- | --- | --- |
| 3.11 | security | 2022-10-24 | 2027-10 |
| 3.12 | security | 2023-10-02 | 2028-10 |
| 3.13 | security | 2024-10-07 | 2029-10 |
| 3.14 | bugfix | 2025-10-07 | 2030-10 |
| 3.15 | bugfix | 2026-10-09 | 2031-10 |

Python 3.10 reached end of life on 2026-10-01 and is not part of the supported range. Python 3.16 has not been released.

## Determine the baseline

Use the lowest version the project declares support for, not the version installed locally and not the newest version in CI.

Read, in order:

1. `[project] requires-python` in `pyproject.toml`.
2. Poetry's `[tool.poetry.dependencies] python`.
3. Trove classifiers such as `Programming Language :: Python :: 3.11`.
4. CI matrix, `tox.ini`, or `noxfile.py`.

Use `.python-version` only as a development hint. It often pins a developer's preferred version, not the support floor. If no floor is declared anywhere, ask instead of guessing.

Normalize the result to a major/minor pair. For Ruff, `3.11` maps to `py311`; patch versions do not change the target.

## Ruff target selection

Ruff recommends `[project] requires-python` over `target-version` when a `pyproject.toml` already exists. If `requires-python` is present, omit `target-version` and let Ruff infer the target. If both are present, `target-version` takes precedence.

- If the project has PEP 621 metadata, rely on `requires-python`.
- If the project has no PEP 621 metadata, set `target-version` to the minimum supported version.
- Never leave a copied `target-version` from another project without checking the floor.
- Ruff's own default target is `py311`. A missing `requires-python` can therefore silently target 3.11.
- `target-version` controls version-aware lint suggestions, not whether a standard-library API exists. Verify stdlib APIs against the documentation for the minimum version.

## Feature matrix

The table is cumulative. A project with a 3.11 floor may use the 3.11 row, but not 3.12+ features.

| Floor | Ruff target | Available at this floor (representative) | Not available; use a compatibility layer |
| --- | --- | --- | --- |
| 3.11 | `py311` | PEP 604 unions, built-in generics, `Self`, `StrEnum`, `TaskGroup`, `tomllib`, `except*`, `datetime.UTC` | PEP 695 generics and type aliases, `typing.override`, `typing.TypeIs`, `typing.ReadOnly`, `warnings.deprecated` |
| 3.12 | `py312` | + PEP 695 syntax, `typing.override`, `itertools.batched`, `Path.walk`, f-string quote reuse | `TypeIs`, `ReadOnly`, `warnings.deprecated`, `copy.replace`, `Path.full_match` |
| 3.13 | `py313` | + `TypeIs`, `ReadOnly`, `warnings.deprecated`, `copy.replace`, `Path.full_match`, `typing.NoDefault` | t-strings, `annotationlib`, `compression.zstd`, `Path.copy`/`move` |
| 3.14 | `py314` | + PEP 649/749 deferred annotations, t-strings, `annotationlib`, `compression.zstd`, `Path.copy`/`move`, `concurrent.interpreters` | `frozendict`, `sentinel`, `TypeForm`, lazy imports, comprehension unpacking |
| 3.15 | `py315` | + `frozendict`, `sentinel`, `TypeForm`, TypedDict typed extra items, lazy imports, comprehension unpacking, UTF-8 as the default encoding | If any supported version is below 3.15, continue to pass `encoding` explicitly for file I/O |

## Compatibility patterns

### Use version guards for APIs, not syntax

`sys.version_info` can select an import or a stdlib API:

```python
import sys

if sys.version_info >= (3, 12):
    from typing import override
else:
    from typing_extensions import override
```

The parser still parses the entire file. Newer syntax such as the `type` statement or PEP 695 generic parameters cannot be hidden in a false branch; it will fail on the older interpreter before the branch runs.

### Centralize backports

Put version-dependent imports in one module, for example `_compat.py`, and import from there:

```python
# _compat.py
import sys

if sys.version_info >= (3, 12):
    from typing import override
else:
    from typing_extensions import override

__all__ = ["override"]
```

Use `typing_extensions` only when the minimum version lacks the feature. Do not use it for features already available at the floor.

### Do not use future annotations

The shared Ruff configuration bans `from __future__ import annotations`. Quote forward references where the runtime needs them:

```python
class Node:
    def link(self, other: "Node") -> None:
        self.other = other
```

Quoting every annotation is not required; quote only references that are not defined at annotation-evaluation time.

### Verify the supported range

- Run tests on the minimum and maximum supported versions.
- Include every version from the project's CI matrix.
- Check dependency metadata for supported Python versions.
- Keep `requires-python`, the CI matrix, and the Ruff target consistent.

## End-of-life maintenance

When a Python version reaches end of life:

1. Remove it from `requires-python`, CI, and the Ruff target.
2. Remove compatibility shims that existed only for that version.
3. Re-run the full test matrix.
4. Update this file with a new verification date.

If a project still declares an end-of-life version, do not silently treat it as supported. Flag it and recommend an upgrade; keep the project's declared floor only if the project explicitly requires it.
