# Git Reference

## Configuration — Set Up Before Anything

```bash
# Essential global config
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.autocrlf input    # Unix/macOS
git config --global core.autocrlf true     # Windows (converts on checkout)
git config --global init.defaultBranch main

# Check config
git config --list --show-origin
```

---

## Line Ending Gotcha (Windows ↔ Unix)

**Most common cross-platform Git issue.** Set this per-repo via `.gitattributes`:

```
# .gitattributes — add to every repo
* text=auto                    # Auto-detect text files
*.sh text eol=lf               # Always LF for shell scripts
*.py text eol=lf               # Always LF for Python
*.bat text eol=crlf            # Always CRLF for batch files
*.ps1 text eol=crlf            # Always CRLF for PowerShell
*.png binary                   # Never touch binary files
*.jpg binary
```

---

## Non-Interactive Git (for Scripts / CI / AI Agents)

```bash
# Disable interactive editors
GIT_EDITOR=true git commit --allow-empty    # Skip editor entirely
git commit -m "message"                      # Always use -m

# Disable color output in scripts
git --no-pager log --oneline --no-color

# Quiet mode (suppress progress)
git clone --quiet https://...
git fetch --quiet

# SSH: disable host key prompts
GIT_SSH_COMMAND="ssh -o StrictHostKeyChecking=no -o BatchMode=yes" git clone ...

# HTTPS: pass credentials without prompt
git clone https://token@github.com/org/repo.git
# Or use credential helper
git config --global credential.helper store

# Unattended merge
git merge --no-edit origin/main             # Accept default merge message
git merge --ff-only origin/main            # Fail if not fast-forward
```

---

## Common Operations

```bash
# Clone
git clone https://github.com/org/repo.git
git clone --depth 1 https://...            # Shallow (faster, no history)
git clone -b branch-name https://...       # Specific branch

# Status / diff
git status
git diff                    # Unstaged changes
git diff --staged           # Staged changes
git diff HEAD~1 HEAD        # Last commit diff

# Stage and commit
git add file.txt
git add -p                  # Interactive patch staging
git commit -m "feat: add X"
git commit --amend --no-edit  # Amend last commit without editing message

# Push / pull
git push origin main
git push --force-with-lease  # Safer than --force (checks for upstream changes)
git pull --rebase            # Preferred: rebase instead of merge commit

# Branches
git checkout -b feature/my-feature    # Create and switch
git switch -c feature/my-feature      # Modern syntax
git branch -d feature/done            # Delete merged branch
git branch -D feature/abandoned       # Force delete

# Remote
git remote -v
git remote add upstream https://...
git fetch --all
git fetch origin --prune              # Remove deleted remote branches
```

---

## Undo Operations

```bash
# Unstage (keep changes)
git restore --staged file.txt    # Modern
git reset HEAD file.txt          # Classic

# Discard working directory changes
git restore file.txt
git checkout -- file.txt         # Classic

# Undo last commit (keep changes staged)
git reset --soft HEAD~1

# Undo last commit (keep changes unstaged)
git reset HEAD~1

# Undo last commit (DISCARD changes — destructive)
git reset --hard HEAD~1

# Revert a commit (creates new commit — safe for shared branches)
git revert abc1234

# Clean untracked files
git clean -fd              # Remove untracked files and dirs
git clean -fdx             # Also remove ignored files
git clean -fdn             # Dry run first ✅
```

---

## Stash

```bash
git stash                       # Stash changes
git stash push -m "wip: login"  # Named stash
git stash list
git stash pop                   # Apply and remove latest
git stash apply stash@{1}       # Apply specific (keep in stash)
git stash drop stash@{1}        # Remove specific
git stash branch new-branch     # Create branch from stash
```

---

## Submodules

```bash
# Clone with submodules
git clone --recurse-submodules https://...

# Initialize after cloning without --recurse-submodules
git submodule update --init --recursive

# Update all submodules to latest
git submodule update --remote --merge
```

---

## Tags

```bash
git tag v1.0.0                              # Lightweight tag
git tag -a v1.0.0 -m "Release 1.0.0"      # Annotated tag (preferred)
git push origin v1.0.0                     # Push specific tag
git push origin --tags                     # Push all tags
git tag -d v1.0.0                          # Delete local
git push origin :refs/tags/v1.0.0         # Delete remote
```

---

## CI/CD Patterns

```bash
# Safe clone in CI (detached HEAD OK)
git clone --depth 1 --branch $BRANCH_NAME https://...

# Configure git for CI bot
git config user.email "ci-bot@company.com"
git config user.name "CI Bot"

# Commit and push in CI
git add -A
git diff --cached --quiet || git commit -m "ci: auto-update [skip ci]"
git push origin HEAD:$BRANCH_NAME

# Check if there are changes to commit
if [ -n "$(git status --porcelain)" ]; then
    git commit -am "ci: update generated files"
fi
```

---

## Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `fatal: not a git repository` | Wrong directory | `cd` to repo root |
| `CRLF will be replaced by LF` | Line ending conversion | Set `.gitattributes` |
| `refusing to merge unrelated histories` | Repos have no common ancestor | `git pull --allow-unrelated-histories` |
| `rejected: non-fast-forward` | Remote has commits you don't | `git pull --rebase` first |
| `nothing to commit, working tree clean` | No changes staged | `git add` first |
| `detached HEAD state` | Checked out commit not branch | `git switch -c new-branch` |
| Permission denied (publickey) | SSH key not added | `ssh-add ~/.ssh/id_ed25519` |
| Large file rejected | File > 100MB | Use Git LFS |
