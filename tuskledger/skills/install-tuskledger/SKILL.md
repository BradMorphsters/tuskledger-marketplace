---
name: install-tuskledger
description: Use this skill when a user wants to install Tusk Ledger (the local-first personal finance app at tuskledger.com) on their machine. Covers the full setup — clone, Python venv, npm install, Plaid keys, first run — and surfaces the project's known footguns. Triggers include "install tuskledger", "set up tuskledger", "I want to try tuskledger", or any variant that involves getting the app running for the first time. Does NOT trigger for routine ops on an already-installed checkout (use the bundled doctor command for those).
---

# Install Tusk Ledger

This skill walks the user through installing Tusk Ledger end-to-end. It is
the same workflow that the `/tuskledger-install` slash command in this
plugin invokes — the slash command is the explicit entry point, this skill
is what gets pulled in when the user describes the task in their own words.

## What Tusk Ledger is (1-line context)

Local-first personal finance app. Plaid → SQLite on the user's laptop →
FastAPI + React UI. MIT licensed. Optional local Ollama for AI Insights
and the Ask panel. No cloud, no telemetry, no SaaS.

## Pre-flight (don't proceed if any fail)

Verify with the user that they have:

- **macOS or Linux** (Windows works via WSL2; native Windows is untested).
- **Python 3.12+** (`python3 --version`).
- **Node 22+** (`node --version`).
- **uvx** for the bundled MCP server (`which uvx`; install via
  `pip install uv` if missing).
- **Disk:** ~500 MB for the repo + venv + node_modules.
- **A Plaid account** (free) at https://dashboard.plaid.com — sandbox keys
  are fine to start; production access takes a few business days. Tell
  them to register before you start the install if they haven't already.

## Steps

### 1. Choose an install path

Default: `~/Documents/tuskledger`. Confirm with the user — some prefer
`~/code/` or `~/projects/` or similar. Use absolute paths from here on.

### 2. Clone

```
git clone https://github.com/BradMorphsters/tuskledger.git <install-path>
cd <install-path>
```

### 3. Backend setup

```
cd backend
python3 -m venv venv
./venv/bin/pip install -r requirements.txt
cp .env.example .env
```

This takes 1-3 minutes. While it runs, tell the user about the next step
so they're ready.

### 4. Plaid keys (REQUIRED — pause here)

Open `backend/.env` in their editor of choice and fill in:

- `PLAID_CLIENT_ID` — from https://dashboard.plaid.com/team/keys
- `PLAID_SECRET` — same page; pick the sandbox secret to start
- `PLAID_ENV` — `sandbox` for testing, `production` once approved

Optional but commonly set:
- `DEV_BYPASS_AUTH=true` — skip the email/password + TOTP MFA login
  flow on a single-user laptop. The app's startup guard refuses to
  start if this is true AND the server is bound to a non-localhost
  interface, so it's safe.
- `LLM_ENABLED=true` — enables AI Insights card + Ask panel.
  Requires Ollama running locally (next section).

**Wait for the user to confirm they've saved .env before continuing.**

### 5. Frontend setup

```
cd <install-path>/frontend
npm install
```

Takes 1-2 minutes.

### 6. First run

```
cd <install-path>
./start.sh
```

Open http://127.0.0.1:5173 in the browser. The user should see either:

- The **Login screen** (if they didn't set `DEV_BYPASS_AUTH=true`), where
  they create their initial email/password + TOTP MFA, OR
- The **Dashboard** directly (if they did set it). Demo mode is on by
  default so it's pre-populated with synthetic data.

### 7. Verify

Run `./tuskledger doctor` from the checkout root. Should report all
checks passing. If the doctor command isn't there yet (older checkout),
the bundled `/tuskledger-doctor` slash command does the equivalent
inline.

## Optional: enable AI Insights + Ask panel

```
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.1:8b
# then in backend/.env:
LLM_ENABLED=true
# restart the backend
```

On Apple Silicon a narrative renders in 5-15 seconds. On Intel, drop to
`phi3:mini` or leave the LLM features off.

## Footguns to mention

- If `pip install` errors on `cffi` / `cryptography` (macOS): the user is
  missing Xcode CLI tools — `xcode-select --install`.
- If port 8000 or 5173 is in use: `lsof -i :8000` to find the conflict.
- The auto-backup script keeps daily SQLite snapshots in `backend/backups/`.
  Grows ~500 KB/day; the user should know but doesn't need to act.
- Plaid free tier: 100 connected items. For one household that's
  permanent. If they exceed it, they pay Plaid directly — Tusk Ledger
  doesn't proxy.

## After install

The MCP server bundled with this plugin is auto-wired — the user's AI
assistant can now call typed tools like `list_accounts`,
`query_transactions`, `get_holdings`, `get_retirement_projection`
against their local data. Try a prompt like "How much did I spend on
groceries last month?" to confirm the wiring.

## What this skill does NOT do

- Migrate from Mint, Empower, or another tool. The CSV import feature
  inside the app handles that — point the user there after install.
- Set up Plaid production access. That's an out-of-band application to
  Plaid, takes a few business days. Tell the user to start with sandbox
  and apply for production in parallel.
- Open ports or expose the app to the public internet. Tusk Ledger is
  designed to stay on the local machine; if the user wants remote
  access, point them at Cloudflare Tunnel rather than poking holes in
  their router.
