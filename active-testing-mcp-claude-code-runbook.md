# Runbook: Connect Claude Code to Active Testing MCP

Replace `<instance>` with your tenant name (for example, `michaelc-lab`).

## Step 1: Confirm MCP is enabled

Log in to `https://<instance>.nonamesec.com/active/overview` and go to **System Settings > Integrations**.

If there's no MCP section, ask Akamai Support to enable the `mcp_enabled` feature flag.

## Step 2: Verify the server responds

```bash
curl -si https://<instance>.nonamesec.com/active/backend/oauth/mcp/rs | grep -i www-authenticate
```

**Expected:** a `401` with `scope="mcp"` and a `resource_metadata="..."` URL.

Check that the hostname in `resource_metadata` exactly matches `<instance>.nonamesec.com`. A mismatch (for example, a missing `-env`) is a server bug for Akamai to fix.

## Step 3: Verify OAuth discovery

```bash
curl -s https://<instance>.nonamesec.com/.well-known/oauth-protected-resource
curl -s https://<instance>.nonamesec.com/.well-known/oauth-authorization-server
```

- The first call should return JSON with `resource` and `authorization_servers`.
- The second should return `authorization_endpoint`, `token_endpoint`, `registration_endpoint`, and `S256` under `code_challenge_methods_supported`.
- A 404 on either means discovery is broken. Stop and contact Akamai.

## Step 4: Add the server to Claude Code

```bash
claude mcp remove active-testing 2>/dev/null
claude mcp add --transport http active-testing https://<instance>.nonamesec.com/active/backend/oauth/mcp/rs --scope user
claude mcp get active-testing
```

## Step 5: Authenticate

1. Start Claude Code with `claude`.
2. Type `/mcp`, select **active-testing**, and choose **Authenticate**.
3. Sign in to Active Testing in the browser and approve.

No API token is needed, because OAuth handles it.

## Step 6: Test with a read-only request

In Claude Code, ask:

```
List my Active Testing applications and their latest scan status.
```

Confirm this works before creating applications or running scans.

## Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| No `www-authenticate` header | Wrong hostname, or MCP isn't enabled |
| `resource_metadata` host differs from the instance | Akamai server misconfiguration |
| 404 on a `.well-known` URL | Discovery endpoint not deployed; contact Akamai |
| Auth succeeded but tools are missing | Run `/mcp` and reconnect, or restart Claude Code |
| Token expired or auth errors later | Run `/mcp`, then **Authenticate** again |
| Need to disconnect | `claude mcp remove active-testing` |

## Claude Desktop (optional)

- **Custom connector:** go to Settings > Connectors > Add custom connector and use the MCP URL. This requires a publicly reachable host and a server that accepts the `https://claude.ai/api/mcp/auth_callback` redirect.
- **Local bridge:** if the connector fails, add this to `~/Library/Application Support/Claude/claude_desktop_config.json`, then quit (Cmd+Q) and reopen Claude Desktop:

```json
{
  "mcpServers": {
    "active-testing": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://<instance>.nonamesec.com/active/backend/oauth/mcp/rs"]
    }
  }
}
```

## Reminders

- The MCP server acts with your Active Testing permissions. An admin login gives the assistant admin power.
- Scan only targets you own or are authorized to test.
- A SaaS scanner can't reach `localhost` on your laptop, so use reachable targets.
- Never paste passwords or tokens into chats or shared docs.
