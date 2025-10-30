# Complete Replication Guide

This guide walks you through replicating this multi-agent orchestration system to your own remote server.

## Prerequisites

### What You Need

1. **Remote Server**:
   - Ubuntu 20.04+ or Debian 11+
   - Minimum 2GB RAM (4GB recommended)
   - SSH access with root or sudo privileges
   - Public IP address
   - Ports accessible: 22 (SSH), plus any service ports you need

2. **Local Machine**:
   - Windows 11 Pro / Linux / macOS
   - SSH client installed
   - Claude Code CLI installed

3. **Accounts**:
   - Claude account with API access (claude.ai)
   - OAuth tokens (access + refresh)

---

## Quick Setup (Recommended)

If you cloned this repository from GitHub:

```bash
# Linux/Mac
./setup_wizard.sh

# Windows (PowerShell)
.\setup_wizard.ps1
```

The wizard will:
1. Ask for your SSH server IP and credentials
2. Test SSH connection
3. Ask for your Claude API token
4. Create config.json
5. Update agent files with your server IP
6. Install Docker, Python, and tools on remote server
7. Create claudedev user
8. Install Claude Code CLI
9. Upload your credentials
10. Test the setup

**Time: ~10 minutes**

Then skip to "Verify Installation" section.

---

## Manual Setup (Step-by-Step)

If you prefer manual setup or want to understand each step:

### Step 1: Prepare Remote Server

```bash
# SSH to your server
ssh root@YOUR_SERVER_IP

# Update system
apt update && apt upgrade -y

# Install Docker
apt install -y docker.io docker-compose
systemctl enable docker
systemctl start docker

# Install Python and tools
apt install -y python3 python3-pip python3-venv jq curl wget git

# Create non-root user for Claude
useradd -m -s /bin/bash claudedev
echo "claudedev:YOUR_SECURE_PASSWORD" | chpasswd
usermod -aG sudo,docker claudedev

# Configure sudo without password
echo "claudedev ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/claudedev
chmod 440 /etc/sudoers.d/claudedev
```

### Step 2: Install Claude Code CLI on Remote

```bash
# Switch to claudedev user
su - claudedev

# Install Claude CLI
curl -fsSL https://claude.ai/install.sh | sh

# Verify installation
claude --version

# Create directories
mkdir -p ~/.claude ~/.claude/agents ~/projects
```

### Step 3: Setup Claude Credentials on Remote

**Get your tokens:**
1. Go to https://claude.ai/settings
2. Find OAuth section
3. Copy access_token and refresh_token

**Upload credentials:**

```bash
# On your local machine, create credentials file
cat > /tmp/credentials.json << 'EOF'
{
  "oauth": {
    "access_token": "at-...",
    "refresh_token": "rt-...",
    "expires_at": "2026-01-01T00:00:00Z"
  }
}
EOF

# Copy to remote server
scp /tmp/credentials.json root@YOUR_SERVER_IP:/tmp/

# On remote server
ssh root@YOUR_SERVER_IP
su - claudedev
mv /tmp/credentials.json ~/.claude/.credentials.json
chmod 600 ~/.claude/.credentials.json
```

### Step 4: Test Remote Claude CLI

```bash
# From local machine
ssh root@YOUR_SERVER_IP 'sudo -u claudedev bash -c "cd ~ && claude -p \"Test: Return hostname\" --dangerously-skip-permissions --max-turns 1"'

# Should return hostname and complete successfully
```

### Step 5: Setup Project Registry

```bash
# On remote server
ssh root@YOUR_SERVER_IP
mkdir -p /root/.claude-projects
touch /root/.claude-projects/registry.jsonl

# Or as claudedev user
su - claudedev
mkdir -p ~/.claude-projects
touch ~/.claude-projects/registry.jsonl
```

### Step 6: Configure Local Environment

**Create config.json locally:**

```json
{
  "server": {
    "ip": "YOUR_SERVER_IP",
    "ssh_user": "root",
    "ssh_port": 22,
    "claude_user": "claudedev"
  },
  "claude": {
    "oauth": {
      "access_token": "at-...",
      "refresh_token": "rt-...",
      "expires_at": "2026-01-01T00:00:00Z"
    }
  }
}
```

**Update agent definitions:**

Replace `188.245.38.217` with your server IP in:
- `.claude/agents/task-developer.md`
- `.claude/agents/tester-agent.md`
- `CLAUDE.md`

```bash
# Linux/Mac
sed -i 's/188\.245\.38\.217/YOUR_SERVER_IP/g' .claude/agents/task-developer.md
sed -i 's/188\.245\.38\.217/YOUR_SERVER_IP/g' .claude/agents/tester-agent.md
sed -i 's/188\.245\.38\.217/YOUR_SERVER_IP/g' CLAUDE.md

# Or manually with your text editor
```

---

## Verify Installation

### Test 1: SSH Connection

```bash
ssh root@YOUR_SERVER_IP 'echo "Connection successful"'
```

Expected: "Connection successful"

### Test 2: Remote Claude CLI

```bash
ssh root@YOUR_SERVER_IP 'sudo -u claudedev bash -c "claude --version"'
```

Expected: Version number displayed

### Test 3: Authentication

```bash
ssh root@YOUR_SERVER_IP 'sudo -u claudedev bash -c "cd ~ && claude -p \"Return the current date\" --dangerously-skip-permissions --max-turns 1"'
```

Expected: Current date returned

### Test 4: Docker Access

```bash
ssh root@YOUR_SERVER_IP 'sudo -u claudedev docker ps'
```

Expected: Docker container list (may be empty)

### Test 5: Multi-Agent Workflow

```bash
# Start Claude Code locally
claude

# Test the system
User: Test the plan-agent with a simple task
```

Expected:
- plan-agent analyzes task
- Provides token estimate
- Creates plan
- Returns structured report

---

## Troubleshooting

### Issue: SSH Connection Fails

**Error:** `Connection refused` or `No route to host`

**Solutions:**
1. Check firewall rules:
   ```bash
   # On remote server
   ufw status
   ufw allow 22/tcp
   ```

2. Verify SSH service:
   ```bash
   systemctl status sshd
   systemctl start sshd
   ```

3. Check server IP:
   ```bash
   # On remote server
   ip addr show
   ```

### Issue: Remote Claude Authentication Fails

**Error:** `Authentication failed` or `OAuth token expired`

**Solutions:**
1. Check credentials file exists:
   ```bash
   ssh root@YOUR_SERVER_IP 'sudo -u claudedev ls -la ~/.claude/.credentials.json'
   ```

2. Verify file permissions:
   ```bash
   ssh root@YOUR_SERVER_IP 'sudo -u claudedev stat ~/.claude/.credentials.json'
   ```
   Should be: `-rw------- (600)`

3. Re-upload credentials:
   - Get new tokens from https://claude.ai/settings
   - Follow Step 3 again

### Issue: Docker Permission Denied

**Error:** `permission denied while trying to connect to the Docker daemon`

**Solutions:**
```bash
# Add claudedev to docker group
ssh root@YOUR_SERVER_IP 'usermod -aG docker claudedev'

# Restart Docker
ssh root@YOUR_SERVER_IP 'systemctl restart docker'

# Test
ssh root@YOUR_SERVER_IP 'sudo -u claudedev docker ps'
```

### Issue: Claude CLI Not Found

**Error:** `claude: command not found`

**Solutions:**
```bash
# Reinstall Claude CLI
ssh root@YOUR_SERVER_IP 'su - claudedev -c "curl -fsSL https://claude.ai/install.sh | sh"'

# Check PATH
ssh root@YOUR_SERVER_IP 'sudo -u claudedev bash -c "echo \$PATH"'

# Claude CLI should be in ~/.local/bin
```

### Issue: Registry Not Found

**Error:** `Registry not found at ~/.claude-projects/registry.jsonl`

**Solutions:**
```bash
# Create registry
ssh root@YOUR_SERVER_IP 'mkdir -p ~/.claude-projects && touch ~/.claude-projects/registry.jsonl'
```

---

## Next Steps

After successful installation:

1. **Deploy Your First Service:**
   ```bash
   claude
   User: Deploy nginx on port 8080
   ```

2. **Test Multi-Agent Workflow:**
   - plan-agent estimates task
   - task-developer executes
   - tester-agent validates

3. **Register Projects:**
   Add your projects to the registry:
   ```bash
   ssh root@YOUR_SERVER_IP 'echo '{"project":"my-app","path":"~/my-app","status":"active","services":["my-app-web"],"added":"2025-10-30"}' >> ~/.claude-projects/registry.jsonl'
   ```

4. **Explore Documentation:**
   - `README.md` - Overview
   - `CLAUDE.md` - Technical reference
   - `QUICK_REPLICATION_CHECKLIST.md` - Fast reference

---

## Security Best Practices

1. **SSH Keys:**
   - Use key-based authentication
   - Disable password authentication:
     ```bash
     # In /etc/ssh/sshd_config
     PasswordAuthentication no
     ```

2. **Firewall:**
   - Only open required ports
   - Use `ufw` or `iptables`

3. **Updates:**
   - Keep system updated:
     ```bash
     apt update && apt upgrade -y
     ```

4. **Credentials:**
   - Never commit config.json
   - Use 600 permissions on credential files
   - Rotate tokens periodically

5. **Docker:**
   - Keep Docker updated
   - Use official images
   - Limit container resources

---

**Setup Complete! 🎉**

Your multi-agent orchestration system is ready to use.

Start with:
```bash
claude
User: Deploy nginx service on port 8080
```
