# Agent Skills

[English](README.md) | [中文（简体）](README.zh-CN.md)

面向 AI 编码代理的可复用 [Agent Skills](https://agentskills.io)。每个技能封装一套聚焦的工作流和参考资料，代理只在任务需要时加载。

## 技能

| 技能 | 描述 |
| --- | --- |
| [`python-coding-standards`](skills/python-coding-standards/) | 使用 Ruff 和一套共享的现代 Python 规范来检查、格式化并审查 Python 代码。需要 Python 3.11+ 和 Ruff。 |

## 安装

### Skills CLI

使用 [Skills CLI](https://github.com/vercel-labs/skills) 安装（需要 Node.js 22.20+）。它会检测已安装的客户端，并把每个技能复制到正确的目录：

```bash
# 安装到当前项目
npx skills add jks15satoshi/agent-skills --skill python-coding-standards

# 全局安装
npx skills add jks15satoshi/agent-skills --skill python-coding-standards -g

# 列出本仓库中可用的技能
npx skills add jks15satoshi/agent-skills --list
```

使用 `-a <agent>` 指定客户端（并用 `-y` 跳过确认提示）。各客户端的 Agent ID 和原生技能目录见[支持的客户端](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)表。

### Python 与 uv

[`add-skills`](https://github.com/ludo-technologies/add-skills) 是一个社区维护的 Python 原生 CLI。用 `uvx` 运行它，无需预先安装：

```bash
# 列出本仓库中可用的技能
uvx add-skills jks15satoshi/agent-skills --list

# 安装到当前项目
uvx add-skills jks15satoshi/agent-skills \
  --skill python-coding-standards \
  -a <agent>

# 全局安装
uvx add-skills jks15satoshi/agent-skills \
  --skill python-coding-standards \
  -g -a <agent>
```

`add-skills` 的 `--agent` 默认值为 `claude-code`；要为其他客户端安装，请显式传入 `-a <agent>`。客户端 ID 见该工具的[支持的客户端](https://github.com/ludo-technologies/add-skills#supported-agents)。`add-skills` 并非由 Agent Skills 项目维护；在敏感环境中使用前请先审查该工具。

### 手动安装

克隆仓库：

```bash
git clone https://github.com/jks15satoshi/agent-skills.git
```

把技能目录复制到你的代理会扫描的技能位置。跨客户端约定：所有项目使用 `~/.agents/skills/`，单个项目内使用 `.agents/skills/`：

```bash
# 在克隆的仓库根目录执行：安装到所有项目
mkdir -p ~/.agents/skills
cp -R skills/python-coding-standards ~/.agents/skills/

# 在你的项目内执行：仅安装到当前项目
mkdir -p .agents/skills
cp -R /path/to/agent-skills/skills/python-coding-standards .agents/skills/
```

`.agents/skills/` 位置见 [Agent Skills 实现指南](https://agentskills.io/client-implementation/adding-skills-support)：安装到该目录的技能对任何扫描它的客户端可见。如果你的客户端不扫描该路径，请使用[支持的客户端](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)表中它的原生技能目录。Skills CLI 会自动处理这一点。

保持技能目录名不变：代理会把目录名用作技能名。

## 使用

代理会根据技能的 `description` 按需加载。发起匹配的任务，或显式点名该技能：

```text
Use python-coding-standards to review src/app/service.py.
```

使用客户端的技能选择器，或直接提到技能名。有些客户端还会把已安装的技能暴露为斜杠命令。

## 校验

用 [skills-ref](https://agentskills.io/specification#validation) 校验你修改过的任何技能：

```bash
uvx --from skills-ref agentskills validate ./skills/python-coding-standards
```

确认 CLI 工具能发现该技能：

```bash
npx -y skills@latest add . --list
uvx add-skills . --list
```

## 贡献

每个技能位于 `skills/<name>/SKILL.md`。新增或编辑技能前请阅读 [`AGENTS.md`](AGENTS.md)。遵循 Agent Skills 的编写指南，让每个技能保持聚焦，并在提交更改前运行上面的校验命令。

## 许可证

本项目采用 [MIT License](LICENSE) 许可。
