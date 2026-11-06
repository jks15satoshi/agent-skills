# Custom Project Standards

This skill ships shared defaults. A project may define its own Python rules in any form and location. There is no required file name, format, or section layout: prose, bullets, a table, or a link to an internal page all work.

**Do not add project rules to this file.** This file is guidance only; keep the rules in the project, outside the installed skill directory.

## Where to put the rules

Use whatever your team already uses for project guidance. Common choices:

- a `## Python Standards` section in `AGENTS.md`
- an existing contribution guide or standards document
- a dedicated file such as `PYTHON_STANDARDS.md` or `.agents/python-standards.md`
- client-specific rule files such as `.cursor/rules/` or `.claude/rules/`

These are examples, not requirements.

## How to make the rules discoverable

If the rules are in `AGENTS.md`, no additional step is needed. If they are in a separate file, reference that file from `AGENTS.md`, or point to the file explicitly in the request.

## How the agent applies them

- Project rules take precedence over the shared defaults where they conflict.
- If a project rule weakens a safety rule, the agent flags the conflict instead of applying it silently. Safety rules include secret handling, exception handling, input validation, timezone-aware datetimes, and test isolation.
- The agent does not write project rules into the installed skill directory.
- Skill updates replace the skill directory; keep project rules in the project and under version control.
