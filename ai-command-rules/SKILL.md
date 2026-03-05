---
name: ai-command-rules
description: "Use this skill whenever Claude needs to write, suggest, or execute shell commands, scripts, or terminal instructions — especially in AI coding contexts (Claude Code, Cursor, Copilot, agentic pipelines). Triggers include bash/shell scripts, SSH remote commands, Docker/container operations, Git commands, Python/virtual environment commands, PowerShell, Windows CMD, and any cross-platform CLI tasks. Must be consulted whenever the user mentions multiple environments (Windows/Linux/Mac, sandbox, venv, PowerShell, WSL, Docker), asks Claude to run or execute commands or write a script, or when Claude is about to emit a command block. Prevents syntax errors, wrong-shell assumptions, and commands that break silently across platforms."
---

# AI Command Rules

A skill for writing precise, environment-aware shell commands and scripts that work correctly the first time across Windows, Linux, macOS, sandboxes, virtual environments, and containers.

## Core Principle

**Always detect the environment before writing a command.** Never assume. A command that works on Linux bash will silently fail or behave differently on Windows CMD, PowerShell, or macOS zsh. When in doubt, ask or emit environment-adaptive commands.

---

## Step 1: Detect the Environment

Before writing any command, identify:

| Signal | How to detect |
|---|---|
| OS | User mentions, file paths (C:\\ vs /), or ask explicitly |
| Shell | bash / zsh / fish / PowerShell / CMD / sh |
| Runtime context | Local terminal, Docker container, CI/CD, sandbox, SSH session |
| Python env | venv, conda, pipenv, poetry, system Python, Docker |
| Privileges | Root/sudo available? Admin on Windows? |

**If uncertain**, emit a detection snippet first:
```bash
# Universal OS+shell detector (paste in terminal first)
echo "OS: $(uname -s 2>/dev/null || echo Windows)"
echo "Shell: $SHELL"
echo "Python: $(python --version 2>&1 || python3 --version 2>&1)"
```

---

## Step 2: Environment-Specific Rules

Read the relevant reference file before emitting commands:

- **Linux / macOS / bash / zsh** → `references/unix.md`
- **Windows CMD / PowerShell / WSL** → `references/windows.md`
- **Docker / containers / sandboxes** → `references/containers.md`
- **Python / virtual environments** → `references/python-env.md`
- **SSH / remote execution** → `references/ssh-remote.md`
- **Git** → `references/git.md`

For cross-platform tasks, read all relevant files.

---

## Step 3: Command Writing Checklist

Before emitting any command block, verify:

### ✅ Shell Compatibility
- [ ] Correct quoting style for target shell (single vs double vs backtick)
- [ ] Path separator correct (`/` vs `\`)
- [ ] Line continuation correct (`\` bash vs `` ` `` PowerShell)
- [ ] Variable syntax correct (`$VAR` bash vs `$env:VAR` PowerShell vs `%VAR%` CMD)
- [ ] Logical operators correct (`&&` / `||` bash vs `-and` / `-or` PowerShell)

### ✅ Error Handling
- [ ] Long scripts use `set -euo pipefail` (bash) or `$ErrorActionPreference = 'Stop'` (PS)
- [ ] Destructive commands (`rm`, `del`, `DROP`) have a dry-run or confirmation step
- [ ] Commands that may fail silently have explicit exit code checks

### ✅ Permissions & Privileges
- [ ] `sudo` / `su` used only when needed; never prefix everything with sudo
- [ ] Windows: flag if Admin PowerShell is required
- [ ] Docker: flag if `--privileged` or volume mounts need host permission

### ✅ Environment Isolation
- [ ] Python commands use the correct interpreter (`python` vs `python3` vs `./venv/bin/python`)
- [ ] npm/node commands run inside the correct project directory
- [ ] Docker commands target the correct container/image name

### ✅ Cross-Platform Output
- [ ] If user may run on multiple OSes, provide alternatives (see formats below)

---

## Step 4: Output Formats

### Single environment — emit one block with a label:
````
```bash  # Linux / macOS
pip3 install -r requirements.txt
```
````

### Cross-platform — emit tabbed alternatives:
````
**Linux / macOS (bash/zsh)**
```bash
python3 -m venv .venv && source .venv/bin/activate
```

**Windows (PowerShell)**
```powershell
python -m venv .venv; .\.venv\Scripts\Activate.ps1
```

**Windows (CMD)**
```cmd
python -m venv .venv && .venv\Scripts\activate.bat
```
````

### Agentic / AI coding context — always add safety guards:
````
```bash
set -euo pipefail          # Exit on error, undefined vars, pipe failures
# ... rest of script
```
````

---

## Common Pitfalls to Avoid

| ❌ Wrong | ✅ Right | Why |
|---|---|---|
| `python script.py` | `python3 script.py` (Unix) | macOS/Linux default `python` may be 2.x |
| `pip install X` | `pip install X --break-system-packages` (system Python) | PEP 668 restriction |
| `rm -rf dist/` | `rm -rf ./dist/` | Protects against accidental root deletion |
| `cd /app && run.sh` | `cd /app && ./run.sh` | PATH may not include `.` |
| `source activate` | `conda activate myenv` | conda >= 4.6 deprecated old syntax |
| `docker run image cmd` | `docker run --rm image cmd` | Avoid orphaned containers |
| `ssh user@host cmd` | `ssh -o StrictHostKeyChecking=no user@host 'cmd'` | Non-interactive SSH needs explicit options |
| `&&` in PowerShell | `;` or `-and` | `&&` not supported in PS < 7 |
| `%VAR%` in PowerShell | `$env:VAR` | Different variable syntax |
| `\n` in Windows paths | `/` or `\\` | Use forward slashes where possible |

---

## Sandbox / Restricted Environment Rules

When Claude itself is executing commands (Claude Code, agentic pipelines, sandboxed VMs):

1. **Probe before assuming** — run `which`, `command -v`, or `Get-Command` to verify tool availability
2. **No interactive prompts** — use `-y`, `--yes`, `--non-interactive`, `-f` flags
3. **Absolute paths preferred** — don't rely on PATH being correct
4. **Temp files** — write to `/tmp` (Linux) or `$env:TEMP` (Windows), not current directory
5. **Network** — check if egress is allowed before running `curl`, `pip install`, `npm install`
6. **No `sudo` in containers** — usually running as root already; `sudo` may not exist

---

## Quick Reference: Shell Detection Snippets

```bash
# Detect if running inside Docker
[ -f /.dockerenv ] && echo "In Docker" || echo "Not Docker"

# Detect OS in bash
case "$(uname -s)" in
  Linux*)   OS=Linux ;;
  Darwin*)  OS=Mac ;;
  CYGWIN*|MINGW*|MSYS*) OS=Windows ;;
esac

# Detect active Python venv
[ -n "$VIRTUAL_ENV" ] && echo "venv: $VIRTUAL_ENV" || echo "No venv active"

# Detect conda env
[ -n "$CONDA_DEFAULT_ENV" ] && echo "conda: $CONDA_DEFAULT_ENV"
```

```powershell
# PowerShell: detect version
$PSVersionTable.PSVersion

# PowerShell: detect if running as Admin
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```
