# Agent Skills

[English](README.md) | **中文（简体）**

面向 AI 编码代理的可复用 [Agent Skills](https://agentskills.io)。每个技能封装一套聚焦的工作流和参考资料，代理只在任务需要时加载。

## 技能

| 技能 | 版本 | 描述 |
| --- | --- | --- |
| [`python-coding-standards`](skills/python-coding-standards/) | [1.1.0](skills/python-coding-standards/CHANGELOG.md) | 使用 Ruff 和一套共享的现代 Python 规范来检查、格式化并审查 Python 代码。需要 Python 3.11+ 和 Ruff。 |

每个技能独立遵循 SemVer。点击版本查看该技能的变更日志；发布标签为 `<skill>-v<version>`。

## 安装

### Skills CLI

使用 [Skills CLI](https://github.com/vercel-labs/skills) 安装（需要 Node.js 22.20+）。它会检测已安装的客户端，并把每个技能安装到正确的目录。安装和更新示例中的 `skill-name` 是占位符，请替换为上表中的技能名：

```bash
# 列出本仓库中可用的技能
npx skills add jks15satoshi/agent-skills --list

# 安装到当前项目
npx skills add jks15satoshi/agent-skills --skill skill-name

# 全局安装
npx skills add jks15satoshi/agent-skills --skill skill-name -g
```

使用 `-a <agent>` 指定客户端（并用 `-y` 跳过确认提示）。各客户端的 Agent ID 和原生技能目录见[支持的客户端](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)表。

### 手动安装

克隆仓库：

```bash
git clone https://github.com/jks15satoshi/agent-skills.git
```

把技能目录复制到你的代理会扫描的技能位置。跨客户端约定：所有项目使用 `~/.agents/skills/`，单个项目内使用 `.agents/skills/`：

```bash
# 在克隆的仓库根目录执行：安装到所有项目
mkdir -p ~/.agents/skills
cp -R skills/skill-name ~/.agents/skills/

# 在你的项目内执行：仅安装到当前项目
mkdir -p .agents/skills
cp -R /path/to/agent-skills/skills/skill-name .agents/skills/
```

`.agents/skills/` 位置见 [Agent Skills 实现指南](https://agentskills.io/client-implementation/adding-skills-support)：安装到该目录的技能对任何扫描它的客户端可见。如果你的客户端不扫描该路径，请使用[支持的客户端](https://github.com/vercel-labs/skills/blob/main/docs/agents.md)表中它的原生技能目录。Skills CLI 会自动处理这一点。

保持技能目录名不变：代理会把目录名用作技能名。

## 更新

Skills CLI 会记录每个技能的来源，可以直接原地更新：

```bash
# 检查是否有更新
npx skills check

# 更新指定技能或全部技能
npx skills update skill-name
npx skills update

# 限定全局或项目范围
npx skills update -g skill-name
npx skills update -p skill-name
```

`npx skills check` 发现更新时会返回非零退出码，可用于 CI。

手动安装时，先更新克隆，再替换技能目录：

```bash
git -C /path/to/agent-skills pull
rm -rf ~/.agents/skills/skill-name
cp -R /path/to/agent-skills/skills/skill-name ~/.agents/skills/
```

如果不想每次复制，可以改成符号链接：

```bash
ln -sfn /path/to/agent-skills/skills/skill-name \
  ~/.agents/skills/skill-name
```

更新只会替换技能目录。项目自定义内容应放在项目内，不要放进已安装的 skill。

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
```

## 贡献

每个技能位于 `skills/<name>/SKILL.md`。新增或编辑技能前请阅读 [`AGENTS.md`](AGENTS.md)。遵循 Agent Skills 的编写指南，让每个技能保持聚焦，并在提交更改前运行上面的校验命令。

## 许可证

本仓库采用 [MIT License](LICENSE) 许可。

每个技能目录内也附带一份副本，以便单独安装技能时许可证随技能一起分发。如果某个技能使用不同的许可证，以该技能目录内的 `LICENSE` 文件为准。
