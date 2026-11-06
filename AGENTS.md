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
5. **Bundle the license.** Copy `LICENSE` into the skill directory and set the `license` field in the frontmatter to match. Clients copy skills out of this repository, so a license file outside the skill directory does not travel with the installed skill.
6. **Test behavior changes.** Use two or three realistic prompts and compare against a baseline: without the skill for a new skill, or the previous version for an existing skill. Keep evaluation workspaces out of the repository.
7. **Validate.** Run the validator for every changed skill and fix every reported problem:

```bash
uvx --from skills-ref agentskills validate ./skills/<skill-name>
```

Replace `<skill-name>` with each skill directory you changed. The `skills-ref` package installs the executable as `agentskills`, not `skills-ref`.

A skill is done only when validation passes.

## Local skill discovery

`.agents/skills/python-coding-standards` is a symlink to `../../skills/python-coding-standards`, so agents working in this repository discover the skill through the cross-client `.agents/skills/` convention. Keep one symlink per skill; do not replace `.agents/skills/` with a directory symlink.

On Windows, Git checks symlinks out as plain files when `core.symlinks` is false (the default). The per-skill link then does not work, but `.agents/skills/` remains a real directory, so other skills can still be installed there. To use the link, enable Developer Mode, set `git config core.symlinks true`, and re-checkout the file.

Do not run `npx skills add` inside this repository; the Skills CLI may replace the link with a real directory. Test installs in a separate clone. If the link is replaced, remove the generated directory and restore it with `git checkout -- .agents/skills/python-coding-standards`.

## Versioning and releases

Each skill is versioned independently with SemVer. The version lives in `metadata.version` in `skills/<name>/SKILL.md` and must be a quoted string. The version covers every file in the skill directory except the changelog history. The repository has no repository-wide version.

### Bump rules

A skill's hard contract is how users install, discover, and run it: the directory name and `name`, the `description` trigger coverage, the `compatibility` requirements, the `license`, the Ruff rules enforced by `assets/pyproject.ruff-template.toml`, and the required invocation and output. Everything else is soft guidance.

| Change | Bump |
| --- | --- |
| Typo, link, formatting, or wording with no change in meaning; changelog history correction | PATCH |
| New or tightened check; new reference, asset, example, or workflow step; broadened `description`; reversal of soft guidance; lower Python/Ruff requirement | MINOR |
| Rename or move the skill; narrowed `description`; higher Python/Ruff requirement; license change; removed or weakened Ruff-enforced rule; changed required invocation or output | MAJOR |

`description` is the discovery contract: broadening is MINOR, narrowing is MAJOR, wording-only is PATCH. A MINOR may surface new violations; say so in the changelog and tell users to re-run `ruff check`. When in doubt, bump higher and explain why in the changelog.

New skills start at `0.1.0`. While on 0.x, a breaking change bumps MINOR; release `1.0.0` when the workflow is stable.

### Every skill change is a release

A PR that changes a skill must, in the same PR:

1. Bump `metadata.version`.
2. Add a dated entry to `skills/<name>/CHANGELOG.md`.
3. Update the version in both READMEs.
4. Run the validator and the evaluation from "Writing skills".
5. Use a Conventional Commit type that matches the change, scoped with the skill name: `feat(python-coding-standards): ...`, `fix(...): ...`, or `feat(...)!:` for MAJOR. Use `docs:` only for repository documentation that users do not install.

Repository-only changes (README, AGENTS.md, CI) do not bump a skill.

### Changelog

`skills/<name>/CHANGELOG.md` follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and is the authoritative history. Newest version first; use `## [1.1.0] - YYYY-MM-DD` headings and the Added, Changed, Deprecated, Removed, Fixed, and Security sections. Write for users: what changed, whether behavior changes, and what to do. Do not add an `[Unreleased]` section because every merge is a release.

### Tag and release

After the PR is merged on `master`:

```bash
git tag -a <skill>-v<version> -m "<skill> <version>"
git push origin <skill>-v<version>
```

Create one GitHub Release per tag. Title it `<skill> v<version>` and use the changelog section for that version as the body, plus a link to the full changelog. Never move or reuse a published tag; publish a new patch instead.

Pin an install with the tagged tree URL:

```bash
npx skills add https://github.com/jks15satoshi/agent-skills/tree/<skill>-v<version>/skills/<skill>
```
