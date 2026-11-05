# AGENTS.md

## Writing skills

Read the relevant Agent Skills "For skill creators" page before starting:

- [Best practices for skill creators](https://agentskills.io/skill-creation/best-practices)
- [Optimizing skill descriptions](https://agentskills.io/skill-creation/optimizing-descriptions)
- [Evaluating skill output quality](https://agentskills.io/skill-creation/evaluating-skills)
- [Using scripts in skills](https://agentskills.io/skill-creation/using-scripts)

Work in this order:

1. **Write the trigger.** In the frontmatter, set `name` to the skill directory name. In `description`, state what the skill does and when to use it, using the concrete tasks and terms a user would type. The description is the only part loaded before the skill, so make it specific.
2. **Draft the workflow.** Make `SKILL.md` a path to a correct result: when to use the skill, the steps in order, the commands to run, and the failure modes worth calling out. Write in the imperative, explain why non-obvious rules exist, and preserve the existing structure and voice when editing.
3. **Push detail down.** Keep `SKILL.md` focused (aim for under 500 lines). Put reference material in `references/` and link each file from `SKILL.md` with a "read when" condition; put templates in `assets/` and deterministic helpers in `scripts/`.
4. **Keep it self-contained.** Every file the skill references must live inside its own directory. Clients install skills individually, so links outside the directory break.
5. **Test behavior changes.** Use two or three realistic prompts and compare against a baseline: without the skill for a new skill, or the previous version for an existing skill. Keep evaluation workspaces out of the repository.
6. **Validate.** Run the validator for every changed skill and fix every reported problem:

```bash
uvx --from skills-ref agentskills validate ./skills/<skill-name>
```

The `skills-ref` package installs the executable as `agentskills`, not `skills-ref`.

A skill is done only when validation passes.