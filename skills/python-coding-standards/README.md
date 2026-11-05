# Python Coding Standards

An [Agent Skill](https://agentskills.io) that gives AI coding agents one consistent way to lint, format, and review Python code. It combines an opinionated Ruff configuration with review guidance for conventions Ruff cannot enforce.

## What it covers

- **Ruff** — shared rule set, formatter, focused fixes, and rule-family explanations.
- **Naming** — consistent vocabulary, collections, booleans, units, and API naming.
- **Type annotations** — complete signatures, built-in generics, `TYPE_CHECKING`, and version-appropriate syntax.
- **Functions and data** — guard clauses, keyword-only booleans, immutable value objects, enums, and async boundaries.
- **Docstrings** — Google sections, argument order, defaults, and attribute documentation.
- **Exceptions** — specific catches, minimal `try` blocks, chained errors, and coded suppressions.
- **Testing** — behavior-focused names, arrange-act-assert, parameterization, and isolation.
- **Security and logging** — no `print`, no hardcoded secrets, no unsafe `eval`/`exec` or deserialization.
- **Python compatibility** — syntax and standard-library availability for all maintained Python versions (currently 3.11 through 3.15).

## Requirements

- Python 3.11 or newer.
- [Ruff](https://docs.astral.sh/ruff/).
- [uv](https://docs.astral.sh/uv/) is recommended for running Ruff and validating the skill.

## Installation

```bash
npx skills add jks15satoshi/agent-skills --skill python-coding-standards
```

For global installs, other installers, and client-specific paths, see the [repository README](../../README.md#installation).

## Usage

The agent loads the skill when a request matches its description. You can also name it explicitly:

```text
Use python-coding-standards to review src/app/service.py and fix the violations.
```

```text
Use python-coding-standards to configure Ruff in this project.
```

```text
Use python-coding-standards to check whether this package still supports Python 3.11.
```

## Workflow

The skill follows five steps:

1. Determine the minimum supported Python version from `pyproject.toml`, Poetry metadata, Trove classifiers, or CI configuration. If no floor is declared, it asks instead of guessing.
2. Merge [`assets/pyproject.ruff-template.toml`](assets/pyproject.ruff-template.toml) into the root `pyproject.toml` without overwriting existing project configuration.
3. Write or review code against the general standards and the Python version matrix.
4. Run Ruff on the files actually changed and fix violations in the code.
5. Verify that tests pass across the supported version range.

## What's included

```text
python-coding-standards/
├── README.md
├── SKILL.md
├── assets/
│   └── pyproject.ruff-template.toml
└── references/
    ├── general-standards.md
    ├── project-specific-standards.md
    ├── ruff-config.md
    └── version-compatibility.md
```

| File | Contents | Read when |
| --- | --- | --- |
| [`SKILL.md`](SKILL.md) | Workflow, commands, gotchas, and validation checklist | Starting any task with this skill |
| [`references/general-standards.md`](references/general-standards.md) | Module organization, nomenclature, typing, functions, data, docstrings, exceptions, testing, security | Writing or reviewing code |
| [`references/version-compatibility.md`](references/version-compatibility.md) | Support matrix, baseline detection, compatibility patterns, end of life | Choosing syntax or setting the floor |
| [`references/ruff-config.md`](references/ruff-config.md) | Rule set map, target version, per-file ignores, template adoption | Configuring or troubleshooting Ruff |
| [`references/project-specific-standards.md`](references/project-specific-standards.md) | Placeholder for project-local conventions | A task touches a project-specific library or API |
| [`assets/pyproject.ruff-template.toml`](assets/pyproject.ruff-template.toml) | Canonical Ruff rule set | Merging or reviewing configuration |

## Ruff configuration

Put the configuration in the project's root `pyproject.toml`. Ruff infers its target from `[project] requires-python`; if the project has no PEP 621 metadata, set `target-version` explicitly. If both are present, `target-version` wins, so keep them consistent.

For a focused change, run Ruff on the touched files:

```bash
uv run ruff check --fix path/to/touched_file.py
uv run ruff format path/to/touched_file.py
```

Use `uvx ruff` when Ruff is not a project dependency, or plain `ruff` when uv is not used. A whole-repository run is reserved for greenfield setup or an explicitly requested cleanup. Review the diff before accepting fixes, and never apply `--unsafe-fixes` without manual inspection.

Do not suppress violations by default. When a suppression is unavoidable, include the rule code and a reason:

```python
assert response.status == 200  # noqa: S101  # guaranteed by client contract
```

## Validation

From the repository root, validate the skill structure:

```bash
uvx --from skills-ref agentskills validate ./skills/python-coding-standards
```

Verify Python changes with Ruff:

```bash
uv run ruff check path/to/touched_file.py
uv run ruff format --check path/to/touched_file.py
```

## License

This Agent Skill is licensed under the [MIT License](../../LICENSE).
