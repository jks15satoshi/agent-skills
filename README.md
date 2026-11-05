# Agent Skills

[English](README.md) | [简体中文](README.zh-CN.md)

Reusable [Agent Skills](https://agentskills.io) for AI coding agents. Each skill packages a focused workflow and reference material that an agent loads only when the task calls for it.

## Skills

| Skill | Description |
| --- | --- |
| [`python-coding-standards`](skills/python-coding-standards/) | Lint, format, and review Python code with Ruff and a shared set of modern Python conventions. Requires Python 3.11+ and Ruff. |

## Installation

### Skills CLI

Install with the [Skills CLI](https://github.com/vercel-labs/skills) (Node.js 22.20+). It detects installed clients and copies each skill into the right directory:

```bash
# Install into the current project
npx skills add jks15satoshi/agent-skills --skill python-coding-standards

# Install globally
npx skills add jks15satoshi/agent-skills --skill python-coding-standards -g

# List the skills available in this repository
npx skills add jks15satoshi/agent-skills --list
```

Pass `-a <agent>` to target a specific client (and `-y` to skip confirmation prompts). Agent IDs and native skill directories are listed in the [supported agents](https://github.com/vercel-labs/skills/blob/main/docs/agents.md) table.

### Python with uv

[`add-skills`](https://github.com/ludo-technologies/add-skills) is a community, Python-native CLI. Run it with `uvx` without installing it:

```bash
# List the skills available in this repository
uvx add-skills jks15satoshi/agent-skills --list

# Install into the current project
uvx add-skills jks15satoshi/agent-skills \
  --skill python-coding-standards \
  -a <agent>

# Install globally
uvx add-skills jks15satoshi/agent-skills \
  --skill python-coding-standards \
  -g -a <agent>
```

`add-skills` defaults `--agent` to `claude-code`; pass `-a <agent>` explicitly to install for another client. See the tool's [supported agents](https://github.com/ludo-technologies/add-skills#supported-agents) for IDs. `add-skills` is not maintained by the Agent Skills project; review the tool before using it in a sensitive environment.

### Manual installation

Clone the repository:

```bash
git clone https://github.com/jks15satoshi/agent-skills.git
```

Copy a skill directory into a skills location your agent discovers. The cross-client convention is `~/.agents/skills/` for all projects and `.agents/skills/` inside a project:

```bash
# From the cloned repository root: install for all projects
mkdir -p ~/.agents/skills
cp -R skills/python-coding-standards ~/.agents/skills/

# From your project: install for this project only
mkdir -p .agents/skills
cp -R /path/to/agent-skills/skills/python-coding-standards .agents/skills/
```

The `.agents/skills/` location is described in the [Agent Skills implementation guide](https://agentskills.io/client-implementation/adding-skills-support): skills installed there are visible to any client that scans it. If your client does not scan that path, use its native skills directory from the [supported agents](https://github.com/vercel-labs/skills/blob/main/docs/agents.md) table. The Skills CLI handles this automatically.

Keep the skill directory name unchanged: agents use it as the skill name.

## Usage

Agents load a skill on demand based on its `description`. Ask for a matching task, or name the skill explicitly:

```text
Use python-coding-standards to review src/app/service.py.
```

Use your client's skill picker or mention the skill by name. Some clients also expose installed skills as slash commands.

## Validation

Validate any skill you change with [skills-ref](https://agentskills.io/specification#validation):

```bash
uvx --from skills-ref agentskills validate ./skills/python-coding-standards
```

Verify that the CLI tools discover the skill:

```bash
npx -y skills@latest add . --list
uvx add-skills . --list
```

## Contributing

Each skill lives under `skills/<name>/SKILL.md`. Read [`AGENTS.md`](AGENTS.md) before adding or editing a skill. Follow the Agent Skills authoring guidance, keep each skill focused, and run the validation command above before submitting a change.

## License

This project is licensed under the [MIT License](LICENSE).
