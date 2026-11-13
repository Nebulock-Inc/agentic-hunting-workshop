# Take-home: connect Claude to your hunts (ATHF MCP server)

Module 4 ran Claude Code against ATHF's MCP server. This is how to do the same on your
own laptop, against your own workspace.

## 1. Install

Nothing new. The workshop install already includes the MCP server:

```bash
pip install 'agentic-threat-hunting-framework[all]'
which athf-mcp    # the server; note this full path for step 2
```

(`[all]` pulls in the `mcp` package. With a plain install, add `[mcp]`.)

## 2. Point your client at your workspace

Run these from your ATHF workspace (the folder where you ran `athf init`), with the venv
active.

**Claude Code:**

```bash
claude mcp add athf -s project -- "$(which athf-mcp)" --workspace "$PWD"
claude        # first start asks you to approve the project's MCP server: approve it
```

That writes a `.mcp.json` in the workspace:

```json
{
  "mcpServers": {
    "athf": {
      "type": "stdio",
      "command": "/full/path/to/.venv/bin/athf-mcp",
      "args": ["--workspace", "/full/path/to/your/workspace"]
    }
  }
}
```

Leave out `-s project` to keep it to yourself instead of in a file you might commit.
ATHF's own docs mention `~/.claude/mcp-servers.json`; Claude Code doesn't read that file.
Use `claude mcp add`.

**Claude Desktop:** add the same server under `mcpServers` in
`~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or
`%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "athf": {
      "command": "/full/path/to/.venv/bin/athf-mcp",
      "args": ["--workspace", "/full/path/to/your/workspace"]
    }
  }
}
```

Use full paths. Claude Desktop doesn't see your venv.

## 3. Check it

- Claude Code: `claude mcp list` → `athf … ✔ Connected`. In a session, `/mcp` lists 21
  `athf_*` tools.
- Ask: *"What have I hunted before?"* You should get your hunts back.

**Tools don't appear?** Reload the client. In Claude Code, run `/mcp` and reconnect, or
quit and restart `claude`. Quit Claude Desktop completely and reopen it. `Pending approval`
in `claude mcp list` means you haven't approved the project server yet.

## 4. What the server gives the assistant

| Tool | Use |
|---|---|
| `athf_hunt_list`, `athf_hunt_get`, `athf_hunt_search` | What have I hunted? Open or search a hunt |
| `athf_hunt_coverage` | ATT&CK coverage by tactic: where are the gaps? |
| `athf_similar` | Have I hunted something like this before? |
| `athf_hunt_new`, `athf_hunt_validate` | Write a hunt, check its format |
| `athf_context` | Environment + past hunts + knowledge for one hunt, tactic or platform |
| `athf_research_*`, `athf_agent_run_*` | Research docs and LLM-backed research / hypotheses (need a provider key) |

The more you write in each hunt's CHECK and KEEP (the SPL you ran, hosts, verdict), the
more the assistant has to work with. It can only remember what you wrote down.

The server only reads and writes your workspace. To let the assistant run searches too,
give it a SIEM connection of its own; ATHF's Splunk quickstart is linked below.

## More

- ATHF README, MCP section: https://github.com/Nebulock-Inc/agentic-threat-hunting-framework#mcp-server-new-in-v0110
- Full tool list: https://github.com/Nebulock-Inc/agentic-threat-hunting-framework/blob/v0.24.0/integrations/MCP_CATALOG.md
- Splunk MCP quickstart: https://github.com/Nebulock-Inc/agentic-threat-hunting-framework/blob/v0.24.0/integrations/quickstart/splunk.md
