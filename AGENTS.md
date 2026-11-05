# AGENTS.md

## Skills

When creating, editing or reviewing skills, follow the Agent Skills "For skill creators" documentation: 

- [Best practices for skill creators](https://agentskills.io/skill-creation/best-practices)
- [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- [Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)
- [Using scripts in skills](https://agentskills.io/skill-creation/using-scripts)

Place each skill at `skills/<name>/SKILL.md` so the Skills CLI and other Agent Skills clients can discover it.

Validate skills after every change with [skills-ref](https://agentskills.io/specification#validation) and fix all reported problems before finishing:

```bash
uvx --from skills-ref agentskills validate ./skills/<skill-name>
```

Replace `<skill-name>` with the directory of each skill you changed. A skill is done only when validation passes. The `skills-ref` package installs the executable as `agentskills`, not `skills-ref`.
