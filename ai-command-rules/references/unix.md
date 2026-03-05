# Unix Reference (Linux / macOS / bash / zsh)

## Shell Shebang — Always Specify

```bash
#!/usr/bin/env bash    # Portable bash (preferred)
#!/bin/bash            # Absolute path (less portable)
#!/bin/sh              # POSIX sh only — avoid bashisms
```

## Script Safety Header

Always start non-trivial scripts with:
```bash
set -euo pipefail
# -e  exit on error
# -u  exit on undefined variable
# -o pipefail  propagate pipe failures
```

## Quoting Rules

```bash
VAR="hello world"
echo "$VAR"          # ✅ Double quotes — expands variables, preserves spaces
echo '$VAR'          # ✅ Single quotes — literal, no expansion
echo $VAR            # ❌ Unquoted — word splits on spaces

# Paths with spaces
cd "/path/with spaces/dir"     # ✅
cd /path/with spaces/dir       # ❌ breaks

# Command substitution — prefer $() over backticks
result=$(command)    # ✅ modern, nestable
result=`command`     # ⚠️ legacy, avoid
```

## Variable Syntax

```bash
NAME="world"
echo "$NAME"                    # simple variable
echo "${NAME}"                  # explicit boundary
echo "${NAME:-default}"         # default if unset
echo "${NAME:?'VAR not set'}"   # exit with error if unset
echo "${#NAME}"                 # string length
echo "${NAME^^}"                # uppercase (bash 4+)
```

## Conditionals

```bash
# File checks
[ -f file ]     # is regular file
[ -d dir ]      # is directory
[ -x script ]   # is executable
[ -z "$VAR" ]   # is empty string
[ -n "$VAR" ]   # is non-empty string

# Numeric
[ "$a" -eq "$b" ]   # equal
[ "$a" -lt "$b" ]   # less than

# String (prefer [[ ]] in bash)
[[ "$a" == "$b" ]]
[[ "$a" =~ ^[0-9]+$ ]]   # regex match
```

## Loops

```bash
# Over files
for f in *.txt; do echo "$f"; done

# Over array
arr=("a" "b" "c")
for item in "${arr[@]}"; do echo "$item"; done

# While read lines
while IFS= read -r line; do
  echo "$line"
done < file.txt

# C-style
for ((i=0; i<10; i++)); do echo $i; done
```

## Error Handling

```bash
# Check exit code
if ! command; then
  echo "Command failed" >&2
  exit 1
fi

# Trap for cleanup
trap 'rm -f /tmp/myfile; echo "Cleaned up"' EXIT ERR

# Log errors to stderr
echo "Error: something went wrong" >&2
```

## File Operations — Safety Rules

```bash
# Always use ./ prefix for local scripts
./myscript.sh

# Dangerous — protect with confirmation
read -p "Delete ./dist? [y/N] " confirm
[[ "$confirm" == "y" ]] && rm -rf ./dist/

# Safe recursive delete pattern
rm -rf "${TARGET_DIR:?'TARGET_DIR not set'}/"

# Copy with progress
rsync -avh --progress src/ dst/
```

## Process & Background Jobs

```bash
# Run in background
command &
PID=$!

# Wait for background job
wait $PID

# Timeout
timeout 30s command

# Kill by name
pkill -f "python my_script.py"
```

## macOS-Specific Notes

```bash
# macOS uses BSD tools — some flags differ from GNU
# GNU: sed -i 's/a/b/' file
# macOS: sed -i '' 's/a/b/' file

# Prefer homebrew GNU tools for consistency
brew install coreutils gnu-sed

# macOS default shell is zsh since Catalina
# zsh is mostly bash-compatible but not identical
# For scripts, use #!/usr/bin/env bash explicitly
```

## Pipe Patterns

```bash
# Check if command output is empty
if [ -z "$(command)" ]; then echo "empty"; fi

# Count lines
command | wc -l

# Filter and process
cat file | grep "pattern" | awk '{print $2}' | sort | uniq

# Parallel execution (GNU parallel)
cat list.txt | parallel -j4 process_item {}
```
