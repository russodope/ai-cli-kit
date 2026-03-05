# Python & Virtual Environments Reference

## The Core Problem

`python` vs `python3` vs `./venv/bin/python` — which one runs?
Answer: it depends on OS, PATH, and whether a venv is active.

**Rule: Always be explicit about which Python you're using.**

---

## Python Interpreter Selection

| Context | Command to use |
|---|---|
| Linux/macOS system | `python3` |
| Windows (from python.org) | `python` |
| Active venv (any OS) | `python` (venv redefines it) |
| Explicit venv (no activate) | `./venv/bin/python` (Unix) or `.\venv\Scripts\python.exe` (Win) |
| Docker container | Depends on image — check with `which python` |

```bash
# Always verify before running
python --version
python3 --version
which python3        # Unix
Get-Command python   # PowerShell
```

---

## venv (Built-in, Recommended)

### Create
```bash
# Unix
python3 -m venv .venv

# Windows PowerShell
python -m venv .venv

# With specific Python version
python3.11 -m venv .venv
```

### Activate
```bash
# Linux / macOS (bash/zsh)
source .venv/bin/activate

# Windows PowerShell
.\.venv\Scripts\Activate.ps1

# Windows CMD
.venv\Scripts\activate.bat

# Git Bash on Windows
source .venv/Scripts/activate
```

### Deactivate
```bash
deactivate    # works on all platforms
```

### Use Without Activating (safer in scripts)
```bash
# Unix
.venv/bin/python script.py
.venv/bin/pip install package

# Windows
.venv\Scripts\python.exe script.py
.venv\Scripts\pip.exe install package
```

### Check if Venv is Active
```bash
echo $VIRTUAL_ENV    # Unix — prints path or empty
$env:VIRTUAL_ENV     # PowerShell
```

---

## pip — Common Pitfalls

```bash
# ❌ May install to wrong Python or system Python
pip install package

# ✅ Install to specific Python
python -m pip install package
python3 -m pip install package

# System Python on modern Linux/macOS (PEP 668)
# Will fail with "externally managed environment"
pip install package --break-system-packages   # Override (use carefully)
# Better: use a venv instead

# Install from requirements
pip install -r requirements.txt

# Upgrade pip first (good practice)
python -m pip install --upgrade pip

# Install in development mode
pip install -e .

# No-cache (fresh install)
pip install --no-cache-dir package

# Offline / air-gapped
pip install --no-index --find-links=/local/wheels package
```

---

## conda / Mamba

```bash
# Detect conda
conda info

# Create env
conda create -n myenv python=3.11
conda create --prefix ./envs python=3.11   # Local prefix (portable)

# Activate
conda activate myenv
conda activate ./envs

# Deactivate
conda deactivate

# Install
conda install numpy
conda install -c conda-forge package

# Export / reproduce
conda env export > environment.yml
conda env create -f environment.yml

# List envs
conda env list

# Remove env
conda env remove -n myenv
```

**conda gotcha:** `source activate` is deprecated since conda 4.6. Use `conda activate`.
**conda gotcha:** conda init modifies shell rc files — if `conda activate` fails, run `conda init bash` first.

---

## Poetry

```bash
# Install Poetry (don't use pip)
curl -sSL https://install.python-poetry.org | python3 -

# Create project
poetry new myproject
poetry init

# Install deps
poetry install
poetry install --no-dev    # Production only

# Add dep
poetry add requests
poetry add --dev pytest

# Run in Poetry venv
poetry run python script.py
poetry run pytest

# Activate shell
poetry shell

# Export to requirements.txt
poetry export -f requirements.txt --output requirements.txt
```

---

## pipenv

```bash
# Install
pip install pipenv

# Create / install
pipenv install
pipenv install requests
pipenv install --dev pytest

# Run
pipenv run python script.py

# Activate
pipenv shell
```

---

## Detecting the Environment in Scripts

```python
import sys
import os

# Which Python is running?
print(sys.executable)      # Full path to interpreter
print(sys.version)

# Is a venv active?
in_venv = sys.prefix != sys.base_prefix
print(f"In venv: {in_venv}")

# Is conda active?
in_conda = os.environ.get('CONDA_DEFAULT_ENV') is not None

# Are we in Docker?
in_docker = os.path.exists('/.dockerenv')
```

---

## Cross-Platform Script Patterns

```bash
# Unix — activate then run
#!/usr/bin/env bash
set -euo pipefail
source "$(dirname "$0")/.venv/bin/activate"
python main.py
```

```powershell
# Windows PowerShell — activate then run
$ErrorActionPreference = 'Stop'
& "$PSScriptRoot\.venv\Scripts\Activate.ps1"
python main.py
```

```python
# Python — use subprocess with explicit interpreter (most portable)
import subprocess, sys
subprocess.run([sys.executable, "other_script.py"], check=True)
```
