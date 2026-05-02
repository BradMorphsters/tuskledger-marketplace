---
description: Guided install of the Tusk Ledger app — clone the repo, set up the venv + npm, walk through Plaid keys, and start the dev servers.
allowed-tools: [bash, read, write, edit]
argument-hint: "[optional install path]"
---

# /tuskledger-install

You are walking the user through installing Tusk Ledger on their machine.
This is a guided install, not an unattended one — pause and report progress
at each step so the user can intervene if something goes sideways.

**Target install path:** `${1:-~/Documents/tuskledger}` (use `$1` if provided
above; default to `~/Documents/tuskledger` otherwise).

## Steps

1. **Confirm prerequisites.** Run these checks (don't proceed if any fail —
   tell the user what's missing and the install command for their OS):
   - `git --version` (any recent version)
   - `python3 --version` (3.12+)
   - `node --version` (22+)
   - `which uvx` (used to run the bundled MCP server)

2. **Clone the repo.** `git clone https://github.com/BradMorphsters/tuskledger.git <target>`.
   If the directory exists, ask before overwriting.

3. **Backend setup.**
   - `cd <target>/backend`
   - `python3 -m venv venv`
   - `./venv/bin/pip install -r requirements.txt`
   - `cp .env.example .env`

4. **Plaid keys — pause here.** Open `<target>/backend/.env` and tell the
   user they need to fill in `PLAID_CLIENT_ID`, `PLAID_SECRET`, and
   `PLAID_ENV` (sandbox to start, production once they've applied for
   their own production access). Link them to https://dashboard.plaid.com.
   Wait for confirmation before continuing.

5. **Frontend setup.** `cd <target>/frontend && npm install`.

6. **First run.**
   - `cd <target> && ./start.sh` (or run backend + frontend in separate
     terminals if start.sh isn't present)
   - Open http://127.0.0.1:5173 in the browser.
   - The user should land on the Login screen (or the Dashboard if
     `DEV_BYPASS_AUTH=true` was set in .env).

7. **Verify.** Run `/tuskledger-doctor` (the sibling command in this
   plugin) to confirm everything's wired up.

## After install

Mention these next steps without doing them:
- The MCP server bundled with this plugin is already wired up — no extra
  config needed for typed AI access to the data.
- Optional: install Ollama (`curl -fsSL https://ollama.com/install.sh | sh`)
  and `ollama pull llama3.1:8b` to enable the AI Insights card and the
  in-app Ask panel. Set `LLM_ENABLED=true` in backend/.env afterward.
- Demo mode is on by default so the user can poke around with synthetic
  data before linking real accounts.

## Footguns to call out

- If `pip install` fails on `cffi` or `cryptography`, the user is missing
  Xcode command-line tools (macOS): `xcode-select --install`.
- If port 8000 or 5173 is already in use, the dev servers will fail
  silently — `lsof -i :8000` to identify the conflict.
- The backups folder grows ~500KB/day. Mention but don't act on it.
