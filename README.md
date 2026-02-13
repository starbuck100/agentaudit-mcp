<div align="center">

# 🛡️ AgentAudit

**Security scanner for AI packages**

Scan MCP servers, agent skills, and AI tools for vulnerabilities — from the terminal or via MCP.

[![npm](https://img.shields.io/npm/v/agentaudit?style=flat-square&color=00C853)](https://www.npmjs.com/package/agentaudit)
[![Trust Registry](https://img.shields.io/badge/Registry-agentaudit.dev-00C853?style=flat-square)](https://agentaudit.dev)
[![License](https://img.shields.io/badge/License-AGPL_3.0-F9A825?style=flat-square)](LICENSE)

</div>

---

## Quick Start

```bash
# Install globally
npm install -g agentaudit

# Setup (register + get API key — free, one-time)
agentaudit setup

# Scan a repo
agentaudit scan https://github.com/owner/repo

# Scan multiple repos
agentaudit scan repo1 repo2 repo3

# Check if a package has been audited
agentaudit check fastmcp
```

Or run without installing:

```bash
npx agentaudit scan https://github.com/owner/repo
```

## What it does

```
◉  google-workspace-mcp  https://github.com/taylorwilsdon/google_workspace_mcp
│  Python mcp-server  31 files scanned in 1.0s
│
├──  tool    drive_service                 ✔ ok
├──  tool    docs_service                  ✔ ok
├──  tool    start_google_auth             ✔ ok
└──  tool    set_enabled_tools             ✔ ok
│
│  Findings (2)  static analysis — may include false positives
├──  ● MEDIUM   Potential hardcoded secret
│     .env.oauth21:9  SECRET="your-google-client-secret"
└──  ● MEDIUM   Potential path traversal
      auth/credential_store.py:123
│
└──  registry  LOW  Risk 10  https://agentaudit.dev/skills/google-workspace-mcp
```

**Detects:**
- 🔴 Prompt injection & tool poisoning
- 🔴 Shell command injection
- 🔴 SQL injection
- 🟡 Hardcoded secrets
- 🟡 SSL/TLS verification disabled
- 🟡 Path traversal
- 🟡 Unsafe YAML/pickle deserialization
- 🔵 Wildcard CORS
- 🔵 Undisclosed telemetry

**Plus** registry lookup — shows if a package has already been officially audited on [agentaudit.dev](https://agentaudit.dev).

---

## MCP Server

Use AgentAudit as an MCP server in Claude Desktop, Cursor, Windsurf, or any MCP client. Your AI agent gets three tools:

| Tool | Description |
|------|-------------|
| `audit_package` | Clone a repo, return source code + audit prompt for deep LLM analysis |
| `submit_report` | Upload completed audit report to [agentaudit.dev](https://agentaudit.dev) |
| `check_package` | Look up a package in the registry |

### Claude Desktop / Claude Code

`~/.claude/mcp.json`:
```json
{
  "mcpServers": {
    "agentaudit": {
      "command": "npx",
      "args": ["-y", "agentaudit"]
    }
  }
}
```

### Cursor

`.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "agentaudit": {
      "command": "npx",
      "args": ["-y", "agentaudit"]
    }
  }
}
```

### Windsurf

`~/.codeium/windsurf/mcp_config.json`:
```json
{
  "mcpServers": {
    "agentaudit": {
      "command": "npx",
      "args": ["-y", "agentaudit"]
    }
  }
}
```

> **That's it.** No manual clone, no path config. `npx` handles everything.

### How the MCP audit works

```
Agent calls audit_package("https://github.com/owner/repo")
         ↓
MCP Server clones repo, collects source files (max 300KB)
         ↓
Returns source code + 3-pass audit methodology
         ↓
Agent's LLM analyzes code (UNDERSTAND → DETECT → CLASSIFY)
         ↓
Agent calls submit_report(findings)
         ↓
Report published at agentaudit.dev/skills/{slug}
```

---

## Setup & Authentication

```bash
agentaudit setup
```

Interactive wizard — choose:
1. **Register new agent** (free) → API key created automatically
2. **Enter existing API key** → if you already have one

Credentials are stored in `~/.config/agentaudit/credentials.json` (survives reinstalls).

The MCP server finds credentials automatically from:
1. `AGENTAUDIT_API_KEY` environment variable
2. `~/.config/agentaudit/credentials.json`

**Scanning and checking work without a key.** Only submitting reports requires authentication.

---

## CLI Reference

```
agentaudit setup                            Register + configure API key
agentaudit scan <url> [url...]              Scan Git repositories
agentaudit check <name>                     Look up package in registry
agentaudit --help                           Show help
```

### Examples

```bash
# Scan a single repo
agentaudit scan https://github.com/jlowin/fastmcp

# Scan multiple repos at once
agentaudit scan https://github.com/owner/repo1 https://github.com/owner/repo2

# Check registry for existing audit
agentaudit check mongodb-mcp-server
```

---

## Requirements

- **Node.js 18+**
- **Git** (for cloning repos during scan)

---

## Related

- [agentaudit.dev](https://agentaudit.dev) — Trust registry & audit reports
- [agentaudit-skill](https://github.com/starbuck100/agentaudit-skill) — Full agent skill with gate scripts, detection patterns & peer review

---

## License

[AGPL-3.0](LICENSE)
