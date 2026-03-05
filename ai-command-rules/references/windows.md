# Windows Reference (CMD / PowerShell / WSL)

## Know Which Shell You're In

| Shell | Identifier | Variable syntax | Line continuation |
|---|---|---|---|
| CMD | `echo %COMSPEC%` → cmd.exe | `%VAR%` | `^` |
| PowerShell 5 | `$PSVersionTable` | `$env:VAR` | `` ` `` (backtick) |
| PowerShell 7+ | version ≥ 7.0 | `$env:VAR` | `` ` `` or `&&` |
| WSL (bash) | `uname -r` shows `-microsoft-` | `$VAR` | `\` |

**Never mix syntax between shells.**

---

## PowerShell — Preferred for Scripting

### Safety Header
```powershell
$ErrorActionPreference = 'Stop'     # Exit on error (like set -e)
Set-StrictMode -Version Latest      # Catch undefined variables (like set -u)
```

### Variables
```powershell
$name = "world"
Write-Host "Hello $name"
Write-Host "Hello ${name}!"         # Explicit boundary

# Environment variables
$env:PATH                           # Read
$env:MY_VAR = "value"              # Set (current session only)
[System.Environment]::SetEnvironmentVariable("MY_VAR","value","User")  # Persistent
```

### Conditionals
```powershell
if ($a -eq $b) { }      # Equal
if ($a -ne $b) { }      # Not equal
if ($a -lt $b) { }      # Less than
if ($a -gt $b) { }      # Greater than
if ($str -like "*.txt") { }      # Wildcard match
if ($str -match "^\d+$") { }     # Regex match
if (Test-Path "file.txt") { }    # File exists
if (Test-Path -PathType Container "dir") { }  # Directory exists
```

### Logical Operators
```powershell
# PowerShell 5 — no && or ||
if ($condition) { cmd1 } else { cmd2 }

# PowerShell 7+ — && and || work
cmd1 && cmd2    # Run cmd2 only if cmd1 succeeds
cmd1 || cmd2    # Run cmd2 only if cmd1 fails

# Chain in PS5
cmd1; if ($LASTEXITCODE -eq 0) { cmd2 }
```

### Loops
```powershell
# ForEach
foreach ($item in $collection) { Write-Host $item }
$collection | ForEach-Object { Write-Host $_ }

# For
for ($i = 0; $i -lt 10; $i++) { Write-Host $i }

# While
while ($condition) { }

# Over files
Get-ChildItem -Filter "*.txt" | ForEach-Object { Write-Host $_.FullName }
```

### Error Handling
```powershell
try {
    risky-command
} catch {
    Write-Error "Failed: $_"
    exit 1
} finally {
    # cleanup always runs
}

# Check exit code of external command
external-tool.exe
if ($LASTEXITCODE -ne 0) {
    Write-Error "Tool failed with code $LASTEXITCODE"
    exit $LASTEXITCODE
}
```

### File Operations
```powershell
# Paths — use forward slashes (works in PS)
$path = "C:/Users/user/project"

# Or join properly
$path = Join-Path $HOME "project" "file.txt"

# Create directory
New-Item -ItemType Directory -Force -Path ".\dist"

# Copy
Copy-Item -Recurse -Force ".\src" ".\dst"

# Delete (with confirmation pattern)
$target = ".\dist"
if (Test-Path $target) {
    Remove-Item -Recurse -Force $target
}

# Read file
Get-Content "file.txt"
Get-Content "file.txt" | ForEach-Object { $_ }
```

### Execution Policy (common blocker)
```powershell
# Check current policy
Get-ExecutionPolicy

# Allow local scripts (run as Admin)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# Run a specific script bypassing policy (no Admin needed)
powershell -ExecutionPolicy Bypass -File .\myscript.ps1
```

### Admin Check
```powershell
$isAdmin = ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
if (-not $isAdmin) {
    Write-Error "This script requires Administrator privileges"
    exit 1
}
```

---

## CMD — Use Only When Necessary

CMD is fragile. Prefer PowerShell. Use CMD only when PS is unavailable.

```cmd
:: Variables
set NAME=world
echo %NAME%

:: Conditionals
if "%NAME%"=="world" (echo yes) else (echo no)
if exist "file.txt" echo file exists
if not exist "dir\" mkdir dir

:: Chain commands
cmd1 && cmd2        :: cmd2 runs only if cmd1 succeeds
cmd1 || cmd2        :: cmd2 runs only if cmd1 fails
cmd1 & cmd2         :: always run both

:: Loops
for %%f in (*.txt) do echo %%f
for /L %%i in (1,1,10) do echo %%i

:: Disable echo in scripts
@echo off

:: Error checking
command
if errorlevel 1 (echo failed & exit /b 1)
```

---

## WSL (Windows Subsystem for Linux)

WSL runs a real Linux kernel — use Unix/bash syntax inside it.

```bash
# Launch WSL from PowerShell/CMD
wsl                        # Default distro
wsl -d Ubuntu              # Specific distro

# Run a single command from PowerShell
wsl bash -c "ls -la /home"

# Access Windows files from WSL
/mnt/c/Users/username/     # C:\ in WSL

# Access WSL files from Windows
\\wsl$\Ubuntu\home\user\   # WSL filesystem in Explorer
```

**Key WSL gotchas:**
- Line endings: Windows files have `\r\n`; run `dos2unix file.sh` before executing
- Permissions: Windows files mounted at `/mnt/c` show as 777 — don't rely on chmod there
- PATH: WSL inherits Windows PATH by default — can cause conflicts

---

## Cross-Platform: PowerShell Core (PS 7+)

PS 7 runs on Linux/macOS too. For cross-platform scripts:

```powershell
# Detect OS in PS
if ($IsWindows) { }
if ($IsLinux)   { }
if ($IsMacOS)   { }

# Path separator
[System.IO.Path]::DirectorySeparatorChar   # \ on Windows, / on Unix

# Temp directory
$env:TEMP      # Windows
$env:TMPDIR    # macOS
"/tmp"         # Linux
[System.IO.Path]::GetTempPath()  # Cross-platform
```
