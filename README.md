# Master Skill - Bot Onboarding & Workspace Setup

## Overview

This master skill defines the standard onboarding process for new bots. It establishes the PAI repository as the central workspace and configures all necessary credentials and dependencies.

**Last Updated**: 2026-03-08  
**PAI Repository**: https://github.com/liyuanWu/PAI

---

## Quick Start

```bash
# One-line onboarding (requires 1Password service token)
./onboard.sh <1password_service_token>
```

---

## Workflow

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  1. 1Password   │────▶│  2. Get GitHub   │────▶│  3. Clone PAI   │
│     Login       │     │    Credentials   │     │    Repository   │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                                                            │
                                                            ▼
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  5. Access All  │◀────│  4. Set PAI as   │◀────│   Verify Setup  │
│  Skills & Agents│     │   Workspace      │     │                 │
└─────────────────┘     └──────────────────┘     └─────────────────┘
```

---

## Step 1: 1Password Authentication

### Prerequisites
- 1Password CLI (`op`) installed
- Service account token provided by user

### Authentication
```bash
# Set service token
export OP_SERVICE_ACCOUNT_TOKEN=<user_provided_token>

# Verify login
op signin --raw
```

### Expected Secrets in 1Password

| Secret | Vault | Item | Field | Purpose |
|--------|-------|------|-------|---------|
| GitHub Username | Coding | Github | `username` | GitHub username (liyuanWu) |
| GitHub PAT | Coding | Github | `password` | GitHub Personal Access Token |
| Supabase URL | Coding | Supabase | `username` | Supabase project URL |
| Supabase Key | Coding | Supabase | `password` | SUPA_SERVICE_ROLE_KEY |

---

## Step 2: Retrieve Credentials

### GitHub Credentials
```bash
# Get GitHub username
GITHUB_USER=$(op read "op://Coding/Github/username")

# Get GitHub PAT
GITHUB_PAT=$(op read "op://Coding/Github/password")

# Verify
echo "GitHub User: $GITHUB_USER"
```

### Supabase Credentials (Optional)
```bash
# Get Supabase URL
SUPA_URL=$(op read "op://Coding/Supabase/username")

# Get Supabase service role key
SUPA_SERVICE_ROLE_KEY=$(op read "op://Coding/Supabase/password")
```

---

## Step 3: Clone PAI Repository

### Clone Command
```bash
# Create workspace directory
mkdir -p /workspace/projects
cd /workspace/projects

# Remove existing PAI folder if present
rm -rf PAI

# Clone using credentials from 1Password
git clone "https://${GITHUB_USER}:${GITHUB_PAT}@github.com/liyuanWu/PAI.git"

# Verify clone
ls -la PAI/
```

### Expected Structure
```
PAI/
├── .git/
├── skills/                    # All skills available here
│   ├── SKILL_SUPABASE.md      # Database operations
│   ├── SKILL_DATA_GOV_HK.md   # Data.gov.hk integration
│   └── SKILL_BUS_QUERY.md     # Bus/transport queries
├── agents/                    # Agent definitions
│   ├── AGENT_CODER.md         # Code development assistant
│   └── AGENT_WEB_RESEARCHER.md # Web research assistant
├── requirements.txt           # Python dependencies
└── README.md
```

---

## Step 4: Set PAI as Workspace

### Environment Setup
```bash
cd /workspace/projects/PAI

# Set workspace environment variable
export COZE_WORKSPACE_PATH=/workspace/projects/PAI

# Add to PYTHONPATH
export PYTHONPATH="${COZE_WORKSPACE_PATH}:${COZE_WORKSPACE_PATH}/src:${PYTHONPATH}"
```

### Verify Workspace
```bash
# Check workspace path
pwd  # Should be /workspace/projects/PAI

# Verify skills are accessible
ls -la skills/
```

---

## Step 5: Access All Skills & Agents

### Available Skills (in PAI/skills/)

| Skill | Description |
|-------|-------------|
| `SKILL_SUPABASE.md` | Supabase database operations (CRUD, table creation) |
| `SKILL_DATA_GOV_HK.md` | Data.gov.hk API integration |
| `SKILL_BUS_QUERY.md` | Bus/transport queries and ETA |

### Available Agents (in PAI/agents/)

| Agent | Description |
|-------|-------------|
| `AGENT_CODER.md` | Code development assistant (Python, JS, Shell) |
| `AGENT_WEB_RESEARCHER.md` | Web research and information gathering |

---

## Complete Onboarding Script

```bash
#!/bin/bash
# onboard.sh - Master onboarding script

set -e

echo "=========================================="
echo "     Bot Onboarding - PAI Workspace"
echo "=========================================="

# Check for service token
if [ -z "$1" ]; then
    echo "Usage: ./onboard.sh <1password_service_token>"
    exit 1
fi

# Step 1: 1Password login
echo ""
echo "[Step 1/5] Authenticating with 1Password..."
export OP_SERVICE_ACCOUNT_TOKEN=$1
op signin --raw > /dev/null
echo "✓ 1Password authenticated"

# Step 2: Get credentials
echo ""
echo "[Step 2/5] Retrieving credentials from Coding vault..."
GITHUB_USER=$(op read "op://Coding/Github/username")
GITHUB_PAT=$(op read "op://Coding/Github/password")
SUPA_URL=$(op read "op://Coding/Supabase/username")
SUPA_SERVICE_ROLE_KEY=$(op read "op://Coding/Supabase/password")
echo "✓ Credentials retrieved"
echo "  - GitHub User: $GITHUB_USER"
echo "  - Supabase URL: $SUPA_URL"

# Step 3: Clone PAI
echo ""
echo "[Step 3/5] Cloning PAI repository..."
cd /workspace/projects
rm -rf PAI  # Remove if exists
git clone "https://${GITHUB_USER}:${GITHUB_PAT}@github.com/liyuanWu/PAI.git"
echo "✓ PAI cloned successfully"

# Step 4: Set workspace
echo ""
echo "[Step 4/5] Setting up workspace..."
cd PAI
export COZE_WORKSPACE_PATH=$(pwd)
export PYTHONPATH="${COZE_WORKSPACE_PATH}:${COZE_WORKSPACE_PATH}/src:${PYTHONPATH}"
echo "✓ Workspace configured: $COZE_WORKSPACE_PATH"

# Step 5: Verify
echo ""
echo "[Step 5/5] Verifying setup..."
SKILL_COUNT=$(ls skills/*.md 2>/dev/null | wc -l)
AGENT_COUNT=$(ls agents/*.md 2>/dev/null | wc -l)
echo "✓ Skills available: $SKILL_COUNT"
echo "✓ Agents available: $AGENT_COUNT"

echo ""
echo "=========================================="
echo "     Onboarding Complete!"
echo "=========================================="
echo "Workspace: $COZE_WORKSPACE_PATH"
echo ""
echo "Available Skills:"
ls -1 skills/*.md 2>/dev/null | sed 's|^|  - |'
echo ""
echo "Available Agents:"
ls -1 agents/*.md 2>/dev/null | sed 's|^|  - |'
```

---

## Usage Examples

### Manual Onboarding
```bash
# Set 1Password token
export OP_SERVICE_ACCOUNT_TOKEN=<token>

# Run onboarding
./onboard.sh $OP_SERVICE_ACCOUNT_TOKEN
```

### Access Skills After Onboarding
```bash
# Read a skill
cat PAI/skills/SKILL_SUPABASE.md

# Read an agent definition
cat PAI/agents/AGENT_CODER.md
```

### Using Skills in Code
```python
import os
os.environ['COZE_WORKSPACE_PATH'] = '/workspace/projects/PAI'

# Follow instructions in PAI/skills/SKILL_SUPABASE.md
# for database operations
```

---

## References

- [1Password Service Accounts](https://developer.1password.com/docs/service-accounts/)
- [GitHub Personal Access Tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
