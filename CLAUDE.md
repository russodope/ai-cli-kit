# AI Command Rules
# Place this file in your project root to apply these rules to Claude Code.
# Source: https://github.com/russodope/claude-skill-command-rules

## Shell Command Policy

Before writing or executing any shell command:

1. **Detect the environment** — OS, shell type, Python env, Docker/container, SSH context
2. **Apply correct syntax** — quoting, variables, operators, path separators differ per shell
3. **Add error handling** — `set -euo pipefail` (bash) or `$ErrorActionPreference = 'Stop'` (PS)
4. **Use non-interactive flags** — `-y`, `--yes`, `-f`, `--non-interactive` in all automated commands
5. **Never assume PATH is correct** — probe with `command -v` or use absolute paths

---

## Environment Detection (run first if unsure)

```bash
echo "OS: $(uname -s 2>/dev/null || echo Windows)"
echo "Shell: $SHELL"
echo "Python: $(python --version 2>&1 || python3 --version 2>&1)"
[ -f /.dockerenv ] && echo "Inside Docker"
[ -n "$VIRTUAL_ENV" ] && echo "venv: $VIRTUAL_ENV"
[ -n "$CONDA_DEFAULT_ENV" ] && echo "conda: $CONDA_DEFAULT_ENV"
```

---

## Rules by Shell

**bash/zsh:**
```bash
set -euo pipefail
python3 -m pip install package          # not: pip install
rm -rf "${DIR:?'DIR not set'}/"         # safe delete
./script.sh                             # not: script.sh
```

**PowerShell:**
```powershell
$ErrorActionPreference = 'Stop'
$env:VAR                                # not: %VAR% or $VAR
python -m pip install package
.\.venv\Scripts\Activate.ps1
```

**Docker:**
```bash
docker run --rm python:3.11-slim cmd    # always --rm, always pin version
docker run --rm -v "$(pwd):/app" img    # bash volume mount
```

**SSH (non-interactive):**
```bash
ssh -o StrictHostKeyChecking=no -o BatchMode=yes -o ConnectTimeout=10 \
    user@host 'bash -s' << 'EOF'
set -euo pipefail
# remote commands here
EOF
```

---

## Hard Rules for Claude Code (Agentic Execution)

- **No interactive prompts** — every command must run to completion without user input
- **No `sudo` in containers** — already root; `sudo` may not exist
- **Temp files** → `/tmp/` not current directory
- **Check network** before any download (`curl -sf URL || exit 1`)
- **Probe before use**: `command -v docker || { echo "docker not found"; exit 1; }`
- **Destructive ops** → dry-run first, then execute with explicit path

---

## Common Mistakes to Avoid

| ❌ | ✅ |
|---|---|
| `python script.py` | `python3 script.py` (Unix) |
| `pip install X` | `python3 -m pip install X` |
| `source activate` | `conda activate envname` |
| `rm -rf dist/` | `rm -rf ./dist/` |
| `docker run img` | `docker run --rm img` |
| `&&` in PowerShell 5 | `;` or `if ($LASTEXITCODE -eq 0)` |
| `ssh host cmd` | `ssh -o BatchMode=yes host 'cmd'` |
| `git commit` (no -m) | `git commit -m "message"` |
