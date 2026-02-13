<div align="center">

# 🛡️ AgentAudit

**Security scanner for AI packages**

Scan MCP servers, agent skills, and AI tools for vulnerabilities.  
MCP server for agents + standalone CLI for humans.

[![npm](https://img.shields.io/npm/v/agentaudit?style=flat-square&color=00C853)](https://www.npmjs.com/package/agentaudit)
[![Registry](https://img.shields.io/badge/Registry-agentaudit.dev-00C853?style=flat-square)](https://agentaudit.dev)
[![License](https://img.shields.io/badge/License-AGPL_3.0-F9A825?style=flat-square)](LICENSE)

</div>

---

## Getting Started

There are two ways to use AgentAudit:

### Option A: MCP Server in your AI editor (recommended)

Add AgentAudit to Claude Desktop, Cursor, or Windsurf. **No API key needed** — your editor's agent runs audits using its own LLM.

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

Then just ask your agent: *"Check which MCP servers I have installed and audit any unaudited ones."*

### Option B: CLI

```bash
# Install
npm install -g agentaudit    # or use npx agentaudit <command>

# 1. Discover your MCP servers
npx agentaudit discover

# 2. Audit unaudited packages (needs an LLM API key)
export ANTHROPIC_API_KEY=sk-ant-...    # or OPENAI_API_KEY=sk-...
npx agentaudit audit https://github.com/owner/repo

# 3. (Optional) Register to upload reports to the public registry
npx agentaudit setup
```

> **Note:** The `audit` command requires an LLM API key (`ANTHROPIC_API_KEY` or `OPENAI_API_KEY`) to analyze code. The `discover`, `scan`, and `check` commands work without one. If you don't have an API key, use `--export` to generate a markdown file you can paste into any LLM, or use AgentAudit as an MCP server (Option A) where no extra key is needed.

### Quick example

```
$ npx agentaudit discover

  AgentAudit v3.3.0
  Security scanner for AI packages

•  Scanning Claude Desktop  ~/.claude/mcp.json    found 2 servers

├──  fastmcp-demo       npm:fastmcp
│    SAFE  Risk 0  ✔ official  https://agentaudit.dev/skills/fastmcp
└──  my-tool            npm:some-mcp-tool
     ⚠ not audited      Run: agentaudit audit https://github.com/user/some-mcp-tool

  Summary  2 servers across 1 config

  ✔  1 audited
  ⚠  1 not audited

  To audit unaudited servers:
  agentaudit audit https://github.com/user/some-mcp-tool  (my-tool)
```

---

## Commands

| Command | What it does | Speed |
|---------|-------------|-------|
| `discover` | Find local MCP servers + check registry | ⚡ instant |
| `check <name>` | Look up a package in the registry | ⚡ instant |
| `scan <url>` | Quick static analysis (regex-based, local) | 🔵 ~2s |
| `audit <url>` | **Deep LLM-powered security audit** | 🔴 ~30s |
| `setup` | Register + configure API key | interactive |

### `scan` vs `audit`

| | `scan` | `audit` |
|--|--------|---------|
| **How** | Regex pattern matching | LLM 3-pass: UNDERSTAND → DETECT → CLASSIFY |
| **Speed** | ~2 seconds | ~30 seconds |
| **Depth** | Surface-level patterns | Semantic code understanding |
| **Needs LLM API key** | No | Yes (`ANTHROPIC_API_KEY` or `OPENAI_API_KEY`) |
| **Uploads to registry** | No | Yes (with `agentaudit setup`) |

### Examples

```bash
# Discover all MCP servers on your machine
npx agentaudit discover

# Quick static scan
npx agentaudit scan https://github.com/owner/repo

# Deep LLM-powered audit
export ANTHROPIC_API_KEY=sk-ant-...
npx agentaudit audit https://github.com/owner/repo

# Export code + audit prompt for manual LLM review
npx agentaudit audit https://github.com/owner/repo --export

# Registry lookup
npx agentaudit check fastmcp

# Register for an API key (free)
npx agentaudit setup
```

---

## MCP Server

Add AgentAudit to your AI editor. Your agent gets 4 tools:

| MCP Tool | Description |
|----------|-------------|
| `discover_servers` | Find all locally installed MCP servers, check registry status |
| `audit_package` | Clone a repo → return source code + audit prompt → you analyze → `submit_report` |
| `submit_report` | Upload your audit report to [agentaudit.dev](https://agentaudit.dev) |
| `check_package` | Quick registry lookup for a package |

### Setup

One-line config — works with `npx`, no manual clone needed:

**Claude Desktop** (`~/.claude/mcp.json`):
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

**Cursor** (`.cursor/mcp.json`):
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

**Windsurf** (`~/.codeium/windsurf/mcp_config.json`):
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

### How an audit works via MCP

```
You: "Audit the blender-mcp server"
                ↓
Agent calls discover_servers → finds blender-mcp
                ↓
Agent calls check_package("blender-mcp") → not in registry
                ↓
Agent calls audit_package("https://github.com/user/blender-mcp")
                ↓
MCP server clones repo, returns source code + audit methodology
                ↓
Agent's LLM analyzes code (3-pass: UNDERSTAND → DETECT → CLASSIFY)
                ↓
Agent calls submit_report(findings_json)
                ↓
Report published at agentaudit.dev/skills/blender-mcp
```

### Authentication

Run `npx agentaudit setup` once. Both CLI and MCP server find credentials automatically:

1. `AGENTAUDIT_API_KEY` environment variable
2. `~/.config/agentaudit/credentials.json` (created by `setup`)

**`discover`, `scan`, and `check` work without a key.** Only `audit`/`submit_report` need one.

---

## What it detects

### Static scan (`scan`)

- 🔴 Prompt injection & tool poisoning
- 🔴 Shell command injection
- 🔴 SQL injection
- 🔴 Unsafe deserialization (pickle, YAML)
- 🟡 Hardcoded secrets
- 🟡 SSL/TLS verification disabled
- 🟡 Path traversal
- 🔵 Wildcard CORS
- 🔵 Undisclosed telemetry

### Deep audit (`audit` / MCP `audit_package`)

Everything above, plus:

- 🔴 Multi-file attack chains (credential harvest → exfiltration)
- 🔴 Agent manipulation (impersonation, capability escalation, jailbreaks)
- 🔴 MCP-specific: tool description injection, resource traversal, unpinned npx
- 🟡 Persistence mechanisms (crontab, shell RC, git hooks, systemd)
- 🟡 Obfuscation (base64 exec, zero-width chars, ANSI escapes)
- 🟡 Context pollution & indirect prompt injection

50+ detection patterns across 8 categories. [Full pattern list →](https://github.com/starbuck100/agentaudit-skill)

---

## Requirements

- **Node.js 18+**
- **Git** (for cloning repos)

---

## Related

- **[agentaudit.dev](https://agentaudit.dev)** — Trust registry with 400+ audit reports
- **[agentaudit-skill](https://github.com/starbuck100/agentaudit-skill)** — Full agent skill with gate scripts, detection patterns & peer review system

---

## License

[AGPL-3.0](LICENSE)
