# Getting started

ASOHawk is a hosted ASO platform your AI agent talks to over [MCP](https://modelcontextprotocol.io). The server is remote HTTP, so there is nothing to install and nothing to run locally: create a key, drop one config block into your client, and ask.

Server URL:

```
https://asohawk.cc/api/mcp
```

## 1. Create a key

Sign up at [asohawk.cc](https://asohawk.cc) (free plan, no credit card). In your workspace, open **Settings → API keys** and create a key. Read by default; add write to let the agent manage tracking and propose App Store changes. The key is shown once, store it like a password.

Every key is scoped: **read** covers every inspection and discovery tool, **write** additionally unlocks tools that change tracking or propose App Store metadata edits. A write tool is invisible to a read-only key, not just refused.

## 2. Connect your client

Replace `ahk_YOUR_KEY` with your real key in any snippet below.

### Claude Code

Run once in any terminal. The `--scope user` flag registers the server for your whole machine, so agents see it from any directory:

```sh
claude mcp add --transport http --scope user asohawk https://asohawk.cc/api/mcp --header "Authorization: Bearer ahk_YOUR_KEY"
```

### Claude Desktop

**Settings → Developer → Edit Config**, then add:

```json
{
  "mcpServers": {
    "asohawk": {
      "type": "http",
      "url": "https://asohawk.cc/api/mcp",
      "headers": {
        "Authorization": "Bearer ahk_YOUR_KEY"
      }
    }
  }
}
```

### Cursor

Save as `.cursor/mcp.json` in your project, or `~/.cursor/mcp.json` to make it available everywhere:

```json
{
  "mcpServers": {
    "asohawk": {
      "url": "https://asohawk.cc/api/mcp",
      "headers": {
        "Authorization": "Bearer ahk_YOUR_KEY"
      }
    }
  }
}
```

### Codex

Add to `~/.codex/config.toml`, the only file Codex reads for MCP servers:

```toml
[mcp_servers.asohawk]
url = "https://asohawk.cc/api/mcp"

[mcp_servers.asohawk.http_headers]
Authorization = "Bearer ahk_YOUR_KEY"
```

To keep the key out of the file, register the server with `codex mcp add` and its `--bearer-token-env-var ASOHAWK_API_KEY` flag, then set that variable in your shell.

### Qwen Code

Add to `~/.qwen/settings.json` (or `.qwen/settings.json` in a project):

```json
{
  "mcpServers": {
    "asohawk": {
      "httpUrl": "https://asohawk.cc/api/mcp",
      "headers": {
        "Authorization": "Bearer ahk_YOUR_KEY"
      }
    }
  }
}
```

### Kimi CLI

```sh
kimi mcp add --transport http asohawk https://asohawk.cc/api/mcp --header "Authorization: Bearer ahk_YOUR_KEY"
```

Configuration lives in `~/.kimi-code/mcp.json` (user level) or `.kimi-code/mcp.json` (project level).

### Gemini CLI

Add to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "asohawk": {
      "httpUrl": "https://asohawk.cc/api/mcp",
      "headers": {
        "Authorization": "Bearer ahk_YOUR_KEY"
      }
    }
  }
}
```

### VS Code

Save as `.vscode/mcp.json` in your workspace:

```json
{
  "servers": {
    "asohawk": {
      "type": "http",
      "url": "https://asohawk.cc/api/mcp",
      "headers": {
        "Authorization": "Bearer ahk_YOUR_KEY"
      }
    }
  }
}
```

### Any other MCP client

Any client that reads an `mcp.json` style config takes the same block as Claude Desktop above. For clients that only speak stdio, bridge with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote):

```json
{
  "mcpServers": {
    "asohawk": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote",
        "https://asohawk.cc/api/mcp",
        "--header", "Authorization: Bearer ahk_YOUR_KEY"
      ]
    }
  }
}
```

## 3. Ask

Real requests to try once your agent is connected:

- Give me a growth report for my apps this week.
- Which of my tracked keywords dropped the most this week?
- Find new keyword opportunities for my app and track the best ones.
- Add "habit tracker" and "daily planner" as keywords for my app.
- Who are my top competitors and which keywords do they rank for that I don't?
- How many downloads and how much revenue does https://apps.apple.com/us/app/duolingo/id570060128 get?

The agent calls ASOHawk and answers in plain language. For longer flows, see the [playbooks](playbooks.md).

## Troubleshooting

**401 Unauthorized.** The key is wrong, revoked, or the header is malformed. The header is exactly `Authorization: Bearer ahk_...`. If the workspace member who created the key is suspended, the key stops working until they are reinstated.

**The agent does not see write tools.** The key is read-scoped. Write tools are invisible to a read-only key by design. Create a key with write scope in **Settings → API keys**.

**`PRECONDITION_FAILED`.** Something the tool needs is missing, most often an App Store Connect or Google Analytics 4 connection. The message says what to connect; the workspace owner does that in **Settings → Integrations**.

**`FORBIDDEN`.** The workspace's agent permissions deny this operation type. See [Permissions and approvals](concepts.md#permissions-and-approvals).

**`RATE_LIMITED` or `UPGRADE_REQUIRED`.** The key hit its rate limit, or the call would exceed the plan's quota. The response says whether a retry can succeed.
