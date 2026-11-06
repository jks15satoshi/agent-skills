# Changelog

All notable changes to `python-coding-standards` are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this skill adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-11-06

### Changed

- Iterator functions now use `iter_*` or `iterate_*` for generic iteration, keep one form per project, and prefer a semantic name (`stream_*`, etc.) when one fits.
- Longer functions now use single blank lines between major phases.
- Data-class docstrings now prefer per-field docs for `@dataclass` where tooling reads them; `Attributes:` remains the portable fallback, with one style per project. The Ruff configuration reference was updated to match.

## [1.0.0] - 2026-11-06

### Added

- Initial release: the shared Ruff rule set and canonical `pyproject.toml` template.
- Python 3.11+ compatibility matrix and baseline detection guidance.
- General standards for naming, typing, functions, data, docstrings, exceptions, testing, security, logging, async, and text I/O.
- Custom project standards guidance: projects can keep rules in `AGENTS.md`, an existing standards document, a dedicated file, or any file the user names; there is no required path or format, and project rules stay outside the installed skill directory.
- A bundled MIT license.

[1.1.0]: https://github.com/jks15satoshi/agent-skills/compare/python-coding-standards-v1.0.0...python-coding-standards-v1.1.0
[1.0.0]: https://github.com/jks15satoshi/agent-skills/releases/tag/python-coding-standards-v1.0.0
