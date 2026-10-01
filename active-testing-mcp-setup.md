# Connecting the Noname `active-testing` MCP server

How to connect Claude Desktop to the Noname Active Testing MCP server on `michaelc-lab.nonamesec.com`, and what to do when it doesn't show up.

## 1. Config entry

File: `~/Library/Application Support/Claude/claude_desktop_config.json`

Add this inside `"mcpServers": { ... }`, alongside `crapi`, `vampi`, etc.:

```json
"active-testing": {
  "command": "/opt/homebrew/bin/npx",
  "args": [
    "-y",
    "mcp-remote",
    "https://michaelc-lab.nonamesec.com/active/backend/oauth/mcp/rs"
  ]
}
```

Notes:
- Separate this entry from its neighbors with commas, and don't put a comma after the last entry. One JSON mistake stops **every** server from loading.
- You don't need `--allow-http` because this endpoint is HTTPS. `crapi` and `vampi` need it because they use plain `http://` on the LAN.

## 2. First-time sign-in (OAuth)

Unlike `crapi` and `vampi`, this server needs a browser login before it will start. Do the sign-in once from a terminal:

```bash
npx -y mcp-remote https://michaelc-lab.nonamesec.com/active/backend/oauth/mcp/rs
```

1. A browser window opens to the Noname login. Sign in.
2. When the terminal shows the connection is established, stop it with `Ctrl+C`.
3. The token is now cached in `~/.mcp-auth`, so Claude Desktop can reuse it.

## 3. Reload Claude Desktop

Quit fully with **Cmd+Q** (closing the window isn't enough), then reopen the app. Look for **`active-testing`** in the tools/connector list. That exact name is what the config uses.

To use it from a claude.ai cloud chat, open that chat in the Claude desktop app on this Mac and send a message. Your local servers come through that link.

## 4. Troubleshooting

| Symptom | Fix |
|---|---|
| `active-testing` missing, others present | Sign-in didn't complete. Check the log (below), then redo step 2. |
| It worked before, now it fails or returns 401 | The cached token is stale. Run `rm -rf ~/.mcp-auth`, redo step 2, then Cmd+Q and reopen. |
| No browser window opens | Run the step 2 command in a terminal and read the error it prints. |
| "registration" / client errors in log | The server may not support dynamic client registration. Check the Noname Active Testing docs for a client ID setting. |
| **All** servers missing | JSON syntax error in the config. Validate it with the command below. |

Log file:

```bash
tail -n 50 ~/Library/Logs/Claude/mcp-server-active-testing.log
```

Validate the config JSON:

```bash
python3 -m json.tool ~/Library/Application\ Support/Claude/claude_desktop_config.json > /dev/null && echo "JSON OK"
```

## Quick reconnect checklist

1. `rm -rf ~/.mcp-auth` (only if it was working before and broke)
2. `npx -y mcp-remote https://michaelc-lab.nonamesec.com/active/backend/oauth/mcp/rs` → sign in → `Ctrl+C`
3. Cmd+Q Claude Desktop → reopen
4. Confirm `active-testing` shows in the tools list
