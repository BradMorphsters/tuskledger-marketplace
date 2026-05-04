---
description: Run the tuskledger doctor health check (Python/Node versions, .env keys, encryption-key permissions, port availability, alembic head, Ollama reachability) and report the findings as actionable items.
allowed-tools: [bash, read]
argument-hint: "[optional path to tuskledger checkout]"
---

# /tuskledger-doctor

Run the project's built-in environment health check and translate the output
into a short list of actionable items. The actual checker is a CLI inside
the repo; this command just invokes it and interprets the result.

## Steps

1. **Locate the checkout.** Try in this order:
   - `${1}` if the user supplied a path argument
   - `~/Documents/tuskledger` (the install default)
   - The current working directory (if it has a `backend/app/main.py`)
   - Otherwise: tell the user `tuskledger doctor` couldn't find an install
     and suggest running `/tuskledger-install` first.

2. **Run the checker.** From the checkout root:
   `./tuskledger doctor --json`

   The CLI emits a stable JSON shape — one entry per check, each with
   `name`, `status` (`ok` | `warn` | `fail`), and `detail`. Catch the
   case where the command isn't there yet (older checkout) and fall
   back to a manual subset:
     - `python3 --version` ≥ 3.12
     - `node --version` ≥ 22
     - `backend/.env` exists and has `PLAID_CLIENT_ID` set
     - `backend/.encryption_key` is chmod 600
     - Port 8000 free (`lsof -i :8000`)
     - Port 3000 free (`lsof -i :3000`)
     - Alembic head matches DB version (`./venv/bin/alembic current`)
     - Ollama reachable at 127.0.0.1:11434 IF `LLM_ENABLED=true` in .env

3. **Output format.** Strict — three sections in this order, omit empty ones:

   **Failing:**
     - One line per failed check with the actionable fix. Example:
       `× Plaid client ID missing — set PLAID_CLIENT_ID in backend/.env`

   **Warnings:**
     - One line per warning with the suggested action. Example:
       `! Ollama not running — AI Insights card will be disabled. Start with: ollama serve`

   **Passing:**
     - Single summary line: `✓ N of M checks passing.`

4. **Don't fix anything automatically.** This command is read-only — it
   surfaces issues, the user (or a follow-up `/tuskledger-install` re-run)
   acts on them.
