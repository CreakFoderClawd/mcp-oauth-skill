---
name: mcp-oauth
description: Connect to OAuth-based MCP servers from OpenClaw using mcporter. Use when integrating external MCP servers that require OAuth authentication (AutoSend, Notion, Linear, Figma, etc.).
---

# MCP OAuth Skill

Connect OpenClaw to OAuth-based MCP servers using [mcporter](https://github.com/nicobailon/mcporter).

> **Status:** 🚧 Work in Progress — documenting setup steps as we go.

## Target MCP Server: AutoSend

**URL:** `https://mcp.autosend.com/`
**Transport:** Streamable HTTP + OAuth 2.0
**Docs:** https://docs.autosend.com/ai/mcp-server

### Available Tools (21 total)

| Category | Tools |
|----------|-------|
| Lists & Segments | `get_lists_and_segments` |
| Templates | `list_templates`, `search_templates`, `get_template`, `create_template`, `update_template`, `delete_template` |
| Senders | `list_senders`, `get_sender` |
| Suppression Groups | `list_suppression_groups`, `get_suppression_group` |
| Campaigns | `list_campaigns`, `get_campaign`, `create_campaign`, `update_campaign`, `delete_campaign`, `duplicate_campaign`, `send_campaign` |
| Analytics | `get_campaign_analytics`, `get_email_activity_analytics` |
| Testing | `send_test_email` |

### Guided Workflows
- `create-campaign` — Step-by-step campaign creation
- `create-template` — Step-by-step template creation

## Prerequisites

- Node.js 18+ (for mcporter)
- OpenClaw configured and running
- AutoSend account (sign up at https://autosend.com)

## Setup Log

_Steps taken during initial setup — this becomes the reference for others._

### Step 1: Create Repository ✅

```bash
gh repo create CreakFoderClawd/mcp-oauth-skill --public --description "OpenClaw skill for connecting OAuth-based MCP servers via mcporter"
```

Created: https://github.com/CreakFoderClawd/mcp-oauth-skill

### Step 2: Install mcporter

```bash
npm install -g mcporter
```

### Step 3: Configure MCP Server

Update `config/mcporter.json`:
```json
{
  "mcpServers": {
    "autosend": {
      "baseUrl": "https://mcp.autosend.com/",
      "auth": "oauth",
      "description": "AutoSend email MCP - campaigns, templates, contacts"
    }
  }
}
```

### Step 4: OAuth Authentication

#### Option A: Desktop/Laptop (has browser)
```bash
npx mcporter auth autosend
# Browser opens → Log in → Authorize → Done
```

#### Option B: Headless Server (human-in-the-loop)

On headless servers, OAuth requires manual intervention:

1. **Agent registers dynamic client & generates auth URL**
2. **Agent sends URL to human**
3. **Human opens URL in browser, authorizes**
4. **Human copies callback URL back to agent**
5. **Agent exchanges code for tokens**

```javascript
// 1. Register dynamic OAuth client
const client = await fetch('https://mcp.autosend.com/oauth/register', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    client_name: "OpenClaw MCP",
    redirect_uris: ["http://127.0.0.1:8765/callback"],
    grant_types: ["authorization_code", "refresh_token"],
    token_endpoint_auth_method: "none"
  })
}).then(r => r.json());

// 2. Generate PKCE challenge
const verifier = crypto.randomBytes(32).toString('base64url');
const challenge = crypto.createHash('sha256').update(verifier).digest('base64url');

// 3. Build auth URL and send to human
const authUrl = `https://autosend.com/oauth/authorize?` + new URLSearchParams({
  client_id: client.client_id,
  redirect_uri: 'http://127.0.0.1:8765/callback',
  response_type: 'code',
  scope: 'mcp:full',
  code_challenge: challenge,
  code_challenge_method: 'S256'
});

// 4. Human authorizes, gets redirected to callback URL (won't load)
// 5. Human shares callback URL: http://127.0.0.1:8765/callback?code=XXX

// 6. Exchange code for tokens
const tokens = await fetch('https://mcp.autosend.com/oauth/token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: new URLSearchParams({
    grant_type: 'authorization_code',
    code: extractedCode,
    redirect_uri: 'http://127.0.0.1:8765/callback',
    client_id: client.client_id,
    code_verifier: verifier
  })
}).then(r => r.json());
```

This pattern works for any OAuth MCP on headless servers where the agent can't open a browser.

### Step 5: Store Tokens for mcporter

After OAuth completes, save tokens for mcporter:

```bash
mkdir -p ~/.mcporter/autosend
# Save the token response to ~/.mcporter/autosend/tokens.json
```

Token format:
```json
{
  "access_token": "eyJhbG...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "ec0b68...",
  "client_id": "...",
  "client_secret": "..."
}
```

### Step 6: Test Tool Calls

Direct curl (with token):
```bash
ACCESS_TOKEN=$(cat ~/.mcporter/autosend/tokens.json | jq -r '.access_token')
curl -s "https://mcp.autosend.com/" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"list_templates","arguments":{}},"id":1}'
```

Via mcporter (once configured):
```bash
npx mcporter call autosend.list_templates
npx mcporter call autosend.list_campaigns
npx mcporter call autosend.get_lists_and_segments
```

## Configuration

### mcporter.json

```json
{
  "mcpServers": {
    "autosend": {
      "baseUrl": "https://mcp.autosend.com/",
      "auth": "oauth",
      "description": "AutoSend email MCP - campaigns, templates, contacts"
    }
  }
}
```

## Helper Script

For headless servers, use the included OAuth helper script:

```bash
# 1. Initialize OAuth (generates auth URL)
node scripts/oauth-helper.js init https://mcp.autosend.com/

# 2. Open the URL in browser, authorize, copy callback URL

# 3. Exchange for tokens
node scripts/oauth-helper.js exchange "http://127.0.0.1:8765/callback?code=XXX&state=YYY"

# 4. Test connection
node scripts/oauth-helper.js test

# 5. Refresh tokens when expired
node scripts/oauth-helper.js refresh
```

## Usage from OpenClaw

Once configured, call MCP tools via:

```bash
npx mcporter call autosend.<tool> <args>
```

Example calls:
```bash
npx mcporter call autosend.list_templates
npx mcporter call autosend.list_senders
npx mcporter call autosend.get_lists_and_segments
npx mcporter call autosend.create_template templateName="Welcome" subject="Hello!" emailTemplate="<html>..."
```

## Troubleshooting

### Token Expired
```bash
node scripts/oauth-helper.js refresh
```

### Invalid Client Credentials
Re-run the full OAuth flow — the client may have been revoked.

### Connection Timeout
Check that `https://mcp.autosend.com/` is reachable and your tokens are valid.

---

## References

- [mcporter GitHub](https://github.com/nicobailon/mcporter)
- [MCP Specification](https://modelcontextprotocol.io/)
- [OpenClaw Skills Guide](https://docs.openclaw.ai)
