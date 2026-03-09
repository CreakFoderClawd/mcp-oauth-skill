---
name: mcp-oauth
description: Connect to OAuth-based MCP servers from OpenClaw using mcporter. Use when integrating external MCP servers that require OAuth authentication (Notion, Linear, Figma, etc.).
---

# MCP OAuth Skill

Connect OpenClaw to OAuth-based MCP servers using [mcporter](https://github.com/nicobailon/mcporter).

> **Status:** 🚧 Work in Progress — documenting setup steps as we go.

## Overview

This skill provides a working pattern for:
1. Installing and configuring mcporter
2. Authenticating with OAuth-based MCP servers
3. Calling MCP tools from OpenClaw

## Prerequisites

- Node.js 18+ (for mcporter)
- OpenClaw configured and running
- Access to an MCP server with OAuth support

## Setup Log

_Steps taken during initial setup — this becomes the reference for others._

### Step 1: Create Repository ✅

```bash
gh repo create CreakFoderClawd/mcp-oauth-skill --public --description "OpenClaw skill for connecting OAuth-based MCP servers via mcporter"
```

Created: https://github.com/CreakFoderClawd/mcp-oauth-skill

### Step 2: Install mcporter

```bash
# TODO: Document installation
npm install -g mcporter
# or
pnpm add -g mcporter
```

### Step 3: Configure MCP Server

```bash
# TODO: Add server config to config/mcporter.json
```

### Step 4: OAuth Authentication

```bash
# TODO: Document OAuth flow
```

### Step 5: Test Tool Calls

```bash
# TODO: Document successful tool calls
```

## Configuration

### mcporter.json

```json
{
  "servers": {
    "your-mcp-server": {
      "baseUrl": "https://your-mcp-server.example.com",
      "auth": "oauth",
      "description": "Your MCP server description"
    }
  }
}
```

## Usage from OpenClaw

Once configured, call MCP tools via:

```bash
npx mcporter call <server>.<tool> <args>
```

## Troubleshooting

_Will be populated as we encounter and solve issues._

---

## References

- [mcporter GitHub](https://github.com/nicobailon/mcporter)
- [MCP Specification](https://modelcontextprotocol.io/)
- [OpenClaw Skills Guide](https://docs.openclaw.ai)
