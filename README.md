# Agent Skills

**English** | [中文（简体）](README.zh-CN.md)

Reusable [Agent Skills](https://agentskills.io) for AI coding agents. Each skill packages a focused workflow and reference material that an agent loads only when the task calls for it.

## Skills

| Skill | Version | Description |
| --- | --- | --- |
| [`python-coding-standards`](skills/python-coding-standards/) | [1.1.0](skills/python-coding-standards/CHANGELOG.md) | Lint, format, and review Python code with Ruff and a shared set of modern Python conventions. Requires Python 3.11+ and Ruff. |

Each skill follows SemVer independently. Select a version to read that skill's changelog; releases are tagged `<skill>-v<version>`.

## Installation

### Skills CLI

Install with the [Skills CLI](https://github.com/vercel-labs/skills) (Node.js 22.20+). It detects installed clients and installs each skill into the right directory. `skill-name` in the installation and update examples is a placeholder for a skill name from the table above:

```bash
# List the skills available in this repository
npx skills add jks15satoshi/agent-skills --list

# Install into the current project
npx skills add jks15satoshi/agent-skills --skill skill-name

# Install globally
npx skills add jks15satoshi/agent-skills --skill skill-name -g
```

Pass `-a <agent>` to target a specific client (and `-y` to skip confirmation prompts). Agent IDs and native skill directories are listed in the [supported agents](https://github.com/vercel-labs/skills/blob/main/docs/agents.md) table.

### Manual installation

Clone the repository:

```bash
git clone https://github.com/jks15satoshi/agent-skills.git
```

Copy a skill directory into a skills location your agent discovers. The cross-client convention is `~/.agents/skills/` for all projects and `.agents/skills/` inside a project:

```bash
# From the cloned repository root: install for all projects
mkdir -p ~/.agents/skills
cp -R skills/skill-name ~/.agents/skills/

# From your project: install for this project only
mkdir -p .agents/skills
cp -R /path/to/agent-skills/skills/skill-name .agents/skills/
```

The `.agents/skills/` location is described in the [Agent Skills implementation guide](https://agentskills.io/client-implementation/adding-skills-support): skills installed there are visible to any client that scans it. If your client does not scan that path, use its native skills directory from the [supported agents](https://github.com/vercel-labs/skills/blob/main/docs/agents.md) table. The Skills CLI handles this automatically.

Keep the skill directory name unchanged: agents use it as the skill name.

## Updating

The Skills CLI records where each skill came from, so you can update in place:

```bash
# Check whether any installed skill has an update
npx skills check

# Update one skill, or every installed skill
npx skills update skill-name
npx skills update

# Limit the update to global or project scope
npx skills update -g skill-name
npx skills update -p skill-name
```

`npx skills check` exits with a non-zero status when an update is available, so it can gate CI.

If you installed manually, update the clone and replace the skill directory:

```bash
git -C /path/to/agent-skills pull
rm -rf ~/.agents/skills/skill-name
cp -R /path/to/agent-skills/skills/skill-name ~/.agents/skills/
```

To avoid repeating the copy, link the clone instead:

```bash
ln -sfn /path/to/agent-skills/skills/skill-name \
  ~/.agents/skills/skill-name
```

Updates replace the skill directory. Keep any project-specific customizations outside the installed skill.

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
```

## Contributing

Each skill lives under `skills/<name>/SKILL.md`. Read [`AGENTS.md`](AGENTS.md) before adding or editing a skill. Follow the Agent Skills authoring guidance, keep each skill focused, and run the validation command above before submitting a change.

## License

This repository is licensed under the [MIT License](LICENSE).

Each skill also bundles a copy in its own directory, so the license travels with the skill when it is installed individually. If a skill ever carries a different license, the `LICENSE` file in that skill's directory takes precedence.
