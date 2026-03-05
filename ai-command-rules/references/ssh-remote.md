# SSH & Remote Execution Reference

## Non-Interactive SSH (for Scripts / AI Agents)

Scripts and AI agents can't respond to prompts. Always use flags that suppress interactive behavior.

```bash
# Basic non-interactive SSH
ssh -o StrictHostKeyChecking=no \
    -o BatchMode=yes \
    -o ConnectTimeout=10 \
    user@host 'command'

# With identity file
ssh -i ~/.ssh/id_ed25519 \
    -o StrictHostKeyChecking=no \
    -o BatchMode=yes \
    user@host 'command'

# Run multi-line script remotely
ssh user@host 'bash -s' << 'EOF'
set -euo pipefail
echo "Running on remote"
ls -la /app
EOF
```

**Key flags:**
| Flag | Purpose |
|---|---|
| `-o StrictHostKeyChecking=no` | Skip host key prompt (use `accept-new` for safer alternative) |
| `-o BatchMode=yes` | Fail instead of prompting for password |
| `-o ConnectTimeout=10` | Don't hang forever |
| `-o ServerAliveInterval=30` | Keep connection alive |
| `-i keyfile` | Explicit identity file |
| `-p 2222` | Custom port |

**Safer alternative to `StrictHostKeyChecking=no`:**
```bash
-o StrictHostKeyChecking=accept-new   # Accept new hosts, reject changed keys
```

---

## Quoting in Remote Commands

**The hardest part of SSH scripting.** Understand two levels of shell expansion:
1. Local shell expands `$VAR` before SSH sends the string
2. Remote shell expands what it receives

```bash
# Expand locally (use double quotes or no quotes)
LOCAL_VAR="hello"
ssh user@host "echo $LOCAL_VAR"         # Sends: echo hello ✅

# Expand remotely (use single quotes or heredoc)
ssh user@host 'echo $HOME'              # Remote $HOME ✅
ssh user@host "echo $HOME"             # Local $HOME (probably wrong) ❌

# Mix: expand some locally, some remotely
ssh user@host "cd /app && echo $LOCAL_VAR && echo \$REMOTE_VAR"
#                                              ↑ escaped = remote

# Safest for complex scripts: heredoc with single-quoted delimiter
ssh user@host 'bash -s' << 'ENDSSH'
# Nothing here expands locally
echo "Remote HOME: $HOME"
ENDSSH

# If you need local variable in heredoc
ssh user@host "bash -s" << ENDSSH   # No quotes on delimiter = local expansion
echo "Local var: $LOCAL_VAR"
echo "Remote home: \$HOME"
ENDSSH
```

---

## SCP / File Transfer

```bash
# Copy file to remote
scp -i ~/.ssh/key file.txt user@host:/remote/path/

# Copy file from remote
scp -i ~/.ssh/key user@host:/remote/file.txt ./local/

# Copy directory
scp -r ./local-dir user@host:/remote/path/

# Better alternative: rsync
rsync -avz --progress ./local/ user@host:/remote/path/
rsync -avz -e "ssh -i ~/.ssh/key -p 2222" ./local/ user@host:/remote/path/

# Dry run first (always for destructive syncs)
rsync -avz --dry-run ./local/ user@host:/remote/path/
```

---

## SSH Tunnels / Port Forwarding

```bash
# Local forward: access remote service locally
# Access remote DB at localhost:5432
ssh -L 5432:localhost:5432 user@host -N

# Remote forward: expose local service on remote
ssh -R 8080:localhost:8080 user@host -N

# Dynamic (SOCKS proxy)
ssh -D 1080 user@host -N

# Background + keepalive
ssh -fN -o ServerAliveInterval=30 \
    -L 5432:localhost:5432 user@host
```

---

## SSH Config File (~/.ssh/config)

Avoid repeating flags by defining hosts:

```
Host myserver
    HostName 192.168.1.100
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    StrictHostKeyChecking accept-new
    ServerAliveInterval 30
    ServerAliveCountMax 3

Host bastion
    HostName bastion.example.com
    User ec2-user
    IdentityFile ~/.ssh/bastion_key

# Jump through bastion to reach internal server
Host internal
    HostName 10.0.0.5
    User ubuntu
    ProxyJump bastion
```

Usage:
```bash
ssh myserver            # Uses config
ssh -J bastion internal # Jump host shorthand
```

---

## Remote Command Patterns for AI Agents

```bash
# Check remote OS before running OS-specific commands
ssh user@host 'uname -s'

# Install package conditionally
ssh user@host 'bash -s' << 'EOF'
if command -v apt-get >/dev/null 2>&1; then
    apt-get install -y package
elif command -v yum >/dev/null 2>&1; then
    yum install -y package
fi
EOF

# Run with timeout (prevent hanging)
timeout 60 ssh user@host 'long-running-command'

# Check exit code explicitly
ssh user@host 'test -f /app/ready'
if [ $? -ne 0 ]; then
    echo "Remote file not ready"
    exit 1
fi
```

---

## Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `Host key verification failed` | Known hosts mismatch | `ssh-keygen -R hostname` then reconnect |
| `Permission denied (publickey)` | Wrong key or no key | `-i` correct keyfile; check `~/.ssh/authorized_keys` on remote |
| `Connection timed out` | Firewall or wrong host | Check port 22 open; use `-p` for custom port |
| `Broken pipe` | Idle connection dropped | Add `ServerAliveInterval 30` |
| `Too many authentication failures` | Too many keys tried | Add `-o IdentitiesOnly=yes -i keyfile` |
| Command not found on remote | PATH differs | Use absolute paths: `/usr/bin/python3` |
| Remote script runs but exits code 1 | Silent failure | Add `set -euo pipefail` and check stderr |
