# ai-command-rules · Claude Skill

> **A Claude skill that writes shell commands correctly the first time — across Windows, Linux, macOS, Docker, SSH, and more.**

[English](#english) · [中文](#中文)

---

## English

### What is this?

`ai-command-rules` is a [Claude Skill](https://support.claude.ai/hc/en-us/articles/27906269580311) that teaches Claude to write precise, environment-aware shell commands and scripts. Instead of generating generic commands that break on your OS or shell, Claude will:

- **Detect your environment first** (OS, shell, Docker, venv, SSH context)
- **Follow per-environment rules** for quoting, variables, error handling, and line endings
- **Emit cross-platform alternatives** when your setup spans multiple environments
- **Apply safety guards** for destructive commands, sandboxes, and CI/CD pipelines

### Covered Environments

| Environment | Reference |
|---|---|
| Linux / macOS (bash, zsh) | `references/unix.md` |
| Windows (PowerShell 5/7, CMD, WSL) | `references/windows.md` |
| Docker / containers / sandboxes | `references/containers.md` |
| Python / venv / conda / poetry | `references/python-env.md` |
| SSH / remote execution | `references/ssh-remote.md` |
| Git | `references/git.md` |

### Installation

1. Download [`ai-command-rules.skill`](./ai-command-rules.skill)
2. Go to [claude.ai](https://claude.ai) → **Settings** → **Skills**
3. Click **Upload Skill** and select the `.skill` file
4. Done — Claude will now automatically apply these rules whenever you ask it to write or run commands

### When does it trigger?

Claude will consult this skill whenever you:
- Ask it to write or run a shell command, script, or terminal instruction
- Mention environments like Windows, Linux, macOS, Docker, SSH, venv, PowerShell, WSL
- Use words like "run", "execute", "script", "deploy", "ssh into", "docker run", etc.

### What's inside

```
ai-command-rules/
├── SKILL.md                  # Main skill — environment detection, checklist, common pitfalls
└── references/
    ├── unix.md               # bash/zsh: quoting, set -euo pipefail, loops, macOS quirks
    ├── windows.md            # PowerShell 5/7, CMD, WSL: variables, error handling, exec policy
    ├── containers.md         # Docker: non-interactive run, compose, sandbox rules, cleanup
    ├── python-env.md         # venv, conda, poetry, pipenv: activation per OS, pip pitfalls
    ├── ssh-remote.md         # Non-interactive SSH, quoting levels, SCP, tunnels, known_hosts
    └── git.md                # Line endings, unattended git, CI/CD patterns, undo operations
```

### Example: before vs after

**Before (without skill):**
```bash
# Claude gives you a generic command
pip install requests
source activate myenv
```

**After (with skill):**
```bash
# Linux / macOS
python3 -m venv .venv && source .venv/bin/activate
python3 -m pip install requests
```
```powershell
# Windows PowerShell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
python -m pip install requests
```

### Contributing

PRs welcome! If you find a missing environment, wrong syntax, or a common pitfall that should be documented, please open an issue or submit a pull request.

---

## 中文

### 这是什么？

`ai-command-rules` 是一个 [Claude Skill（技能包）](https://support.claude.ai/hc/en-us/articles/27906269580311)，让 Claude 在生成 shell 命令和脚本时更加准确——无论你在 Windows、Linux、macOS、Docker、SSH 还是虚拟环境中工作，都能一次生成正确的命令，不再反复修改。

安装后，Claude 会：

- **先检测你的环境**（操作系统、Shell 类型、Docker、venv、SSH 上下文）
- **按环境套用专属规则**（引号写法、变量语法、错误处理、换行符）
- **多环境时给出平行写法**（同时提供 bash 和 PowerShell 版本）
- **自动加安全防护**（破坏性命令确认、沙箱/CI 无交互模式）

### 覆盖的环境

| 环境 | 参考文件 |
|---|---|
| Linux / macOS（bash、zsh） | `references/unix.md` |
| Windows（PowerShell 5/7、CMD、WSL） | `references/windows.md` |
| Docker / 容器 / 沙箱 | `references/containers.md` |
| Python / venv / conda / poetry | `references/python-env.md` |
| SSH / 远程执行 | `references/ssh-remote.md` |
| Git | `references/git.md` |

### 安装方法

1. 下载 [`ai-command-rules.skill`](./ai-command-rules.skill) 文件
2. 打开 [claude.ai](https://claude.ai) → **Settings（设置）** → **Skills（技能）**
3. 点击 **Upload Skill（上传技能）**，选择下载的 `.skill` 文件
4. 完成！之后 Claude 在生成命令时会自动应用这些规范

### 什么时候会触发？

当你：
- 让 Claude 写或执行 shell 命令、脚本、终端指令时
- 提到 Windows、Linux、macOS、Docker、SSH、venv、PowerShell、WSL 等关键词时
- 使用"运行"、"执行"、"脚本"、"部署"、"远程"、"docker run"等表达时

### 文件结构

```
ai-command-rules/
├── SKILL.md                  # 主文件：环境检测、命令编写 checklist、常见错误对照
└── references/
    ├── unix.md               # bash/zsh：引号规则、set -euo pipefail、macOS 差异
    ├── windows.md            # PowerShell 5/7、CMD、WSL：变量、错误处理、执行策略
    ├── containers.md         # Docker：非交互执行、compose、沙箱规则、清理
    ├── python-env.md         # venv/conda/poetry/pipenv：各 OS 激活方式、pip 坑
    ├── ssh-remote.md         # 非交互 SSH、两级引号、SCP/rsync、端口转发
    └── git.md                # 换行符、无人值守 git、CI/CD 模式、撤销操作
```

### 示例：有无 skill 的对比

**没有 skill 时（Claude 给出泛用命令）：**
```bash
pip install requests
source activate myenv
```

**安装 skill 后（Claude 按环境给出准确命令）：**
```bash
# Linux / macOS
python3 -m venv .venv && source .venv/bin/activate
python3 -m pip install requests
```
```powershell
# Windows PowerShell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
python -m pip install requests
```

### 参与贡献

欢迎提 PR！如果你发现有遗漏的环境、错误的语法，或者某个常见坑没有收录，请提 Issue 或直接提交 Pull Request。

---

## License

MIT
