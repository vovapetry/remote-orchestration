# Multi-Agent Orchestration System

> **One-Command Setup**: Clone from GitHub → Run wizard → Start working in 10 minutes

A sophisticated orchestration system for managing remote servers using Claude Code CLI with specialized agents for planning, development, and testing.

[![Setup Time](https://img.shields.io/badge/Setup-10%20min-brightgreen)](QUICK_REPLICATION_CHECKLIST.md)
[![Automation](https://img.shields.io/badge/Automation-Multi--Agent-blue)](.claude/agents)
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey)](README.md)

---

## 🎯 What Is This?

This project enables you to:
- **Deploy services** on remote servers using AI agents
- **Automatically plan, implement, and test** changes through iterative workflow
- **Leverage Claude Code CLI** on remote servers via SSH
- **Track complex tasks** through multi-iteration development cycles
- **Test implementations** using browser automation, API calls, or CLI commands

## 🚀 Quick Start (GitHub Clone Workflow)

### 1. Clone Repository

```bash
git clone https://github.com/vovapetry/remote-orchestration.git
cd remote-orchestration
```

### 2. Run Setup Wizard

**Linux/Mac:**
```bash
./setup_wizard.sh
```

**Windows (PowerShell):**
```powershell
.\setup_wizard.ps1
```

### 3. Enter Your Information

The wizard will ask for:
- SSH server IP address
- SSH credentials
- Claude API token ([Get from here](https://claude.ai/settings))

### 4. Start Working

```bash
claude
```

**That's it! Total time: 10 minutes**

---

## 🏗️ Architecture

```
Local Machine (Windows/Linux/Mac)  │  Remote Server (Ubuntu)
┌─────────────────────────────┐   │  ┌──────────────────────────┐
│  Orchestrator (You)         │ SSH│  │  Claude Code CLI         │
│  ├─ plan-agent             │───┼─▶│  (claudedev user)        │
│  ├─ task-developer         │   │  │                          │
│  └─ tester-agent           │   │  │  Docker Services         │
│                             │   │  │  ├─ Service 1            │
│  Multi-Agent Workflow       │   │  │  ├─ Service 2            │
│  └─ Iteration Tracking      │   │  │  └─ Service N            │
└─────────────────────────────┘   │  └──────────────────────────┘
```

### The Three Agents

1. **plan-agent** - Analyzes tasks, estimates tokens (5-factor model), auto-splits large tasks
2. **task-developer** - Connects to remote Claude CLI, executes plans via SSH
3. **tester-agent** - Tests with browser (Playwright), API (curl), CLI (docker/ssh), or database queries

---

## 📁 Project Structure

```
remote-orchestration/
├── setup_wizard.sh          ⭐ Run this first (Linux/Mac)
├── setup_wizard.ps1         ⭐ Run this first (Windows)
│
├── .claude/agents/          🤖 Agent definitions
│   ├── plan-agent.md        → Planning & token estimation
│   ├── task-developer.md    → Remote execution via SSH
│   └── tester-agent.md      → Testing & validation
│
├── README.md                📖 This file
├── CLAUDE.md                📚 Technical reference
├── Documentation/           📋 Setup and replication guides
│
└── config.template.json     🔧 Configuration template
```

---

## 🔧 Setup Time: ~10 Minutes

The setup wizard automatically:
1. Collects your SSH server info & Claude API token
2. Tests connections
3. Installs Docker, Python, Claude CLI on remote server
4. Creates users & configures permissions
5. Validates the setup

**Total time: ~10 minutes**

---

## 📚 Documentation

- **Quick Start**: You're reading it
- **CLAUDE.md**: Technical reference for Claude Code
- **Guides**: See REPLICATION_GUIDE.md and QUICK_REPLICATION_CHECKLIST.md

---

## 🔐 Security

- ✅ SSH key-based authentication
- ✅ Credentials never committed (`.gitignore`)
- ✅ Docker isolation for services
- ✅ Non-root automation user
- ✅ Secure credential storage

---

## 📝 License

MIT License

---

**Repository:** https://github.com/vovapetry/remote-orchestration

**Version:** 1.0  
**Status:** Production ready  
**Last Updated:** 2025-10-30
