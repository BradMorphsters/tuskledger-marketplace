# tuskledger-marketplace

A Cowork / Claude Code plugin marketplace for [Tusk Ledger](https://www.tuskledger.com)
— the local-first personal finance app.

This marketplace currently ships one plugin: **tuskledger**.

## What you get when you install the plugin

- **`/tuskledger-install`** — guided slash command that walks through
  cloning the repo, setting up the Python venv + npm, configuring Plaid
  keys, and starting the dev servers. Pauses at each meaningful step so
  you can intervene.
- **`/tuskledger-doctor`** — runs the project's built-in environment
  health check and translates the JSON output into a short list of
  actionable items.
- **`install-tuskledger` skill** — auto-invoked when you describe wanting
  to install Tusk Ledger in your own words ("set this up for me", "I want
  to try tuskledger").
- **MCP server (auto-wired)** — bundles the `tuskledger-mcp` PyPI
  package so your AI assistant gets typed access to your local Tusk
  Ledger data via 23 tools (20 read + 3 write) (`list_accounts`,
  `query_transactions`, `get_holdings`, `get_retirement_projection`,
  and more). No cloud, no proxy, no copying secrets into chat. Runs
  locally over stdio.

## Installing the plugin

In Cowork (or any Claude Code-compatible client):

```
/plugin marketplace add BradMorphsters/tuskledger-marketplace
/plugin install tuskledger@tuskledger-marketplace
```

After the install, the MCP server auto-starts when you next launch the
client. Run `/tuskledger-install` to set up the actual app, or just say
"install tuskledger" and the bundled skill picks it up.

## Companion repos

- **App:** https://github.com/BradMorphsters/tuskledger
- **MCP server (the package this plugin's `.mcp.json` references):** https://github.com/BradMorphsters/tuskledger-mcp
- **Marketing site:** https://www.tuskledger.com

## Layout

```
.
├── .claude-plugin/
│   └── marketplace.json    # marketplace manifest (lists plugins)
├── tuskledger/             # the plugin
│   ├── .claude-plugin/
│   │   └── plugin.json     # plugin manifest
│   ├── .mcp.json           # MCP server config (uvx tuskledger-mcp)
│   ├── commands/
│   │   ├── tuskledger-install.md
│   │   └── tuskledger-doctor.md
│   └── skills/
│       └── install-tuskledger/
│           └── SKILL.md
└── README.md
```

## License

MIT — same as the rest of the Tusk Ledger project.
