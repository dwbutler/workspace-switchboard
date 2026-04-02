# TOOLS.md - Platform Notes

Local notes for Switchboard's operational context. Update as the platform grows.

## Model Config

- **Default:** Local Ollama (llama3.2 or equivalent) — zero cost, no data leaves
- **BYOK:** Anthropic / OpenAI (for synthesis passes and complex reasoning)
- Model router lives in `packages/core/src/model/router.ts`

## Switchboard Core

- **Audit flows:** `packages/core/src/audit/`
- **KB builder:** `packages/core/src/kb/`
- **Workspace scaffold:** `packages/core/src/scaffold/`
- **OTP auth:** `packages/core/src/auth/otp.ts`

## Telegram Bot

- **Bot token:** set via `TELEGRAM_BOT_TOKEN` env var (never hardcoded)
- **Webhook server:** `apps/switchboard-bot/`
- **Handler:** `packages/bot/src/handler.ts`

## Provisioning Hooks

When provisioning a new agent:
1. Synthesize audit → KB entries (`KBSynthesizer.synthesize()`)
2. Scaffold workspace (`WorkspaceScaffolder.scaffold()`)
3. Write files to output dir (`KBWriter.write()`)
4. Return `openclaw.json.template` path to user

## Filesystem Constraints

**Two modes: local and deployed.**

### Local (development)
Normal filesystem access. The `drop/` directory is just a local folder (`./drop` by default).
Run it, test it, inspect outputs. No sandbox constraints.

### Deployed (production container)
Read-only filesystem with one exception: a write-only `drop` directory mounted into the container.

- The agent **cannot read from disk** (no session state, no cached data, no config files)
- The agent **can write to `drop/`** — but cannot read back what it wrote
- Write-only mount: outputs land there, the operator picks them up from outside
- Conversation state is in-memory only — ephemeral, isolated per session
- The agent structurally cannot accumulate or exfiltrate user data

**What goes into `drop/`:**
- Provisioned workspace outputs (`.switchboard/` directories, `openclaw.json.template`)
- Audit reports
- Anything the user or operator needs to retrieve after the session

**Code contract:**
- `KBWriter.write()` targets the drop path (env var: `DROP_PATH`, default `./drop`)
- No config files on disk — all config via env vars
- Audit session state is ephemeral; if a user drops off, they start over

## Credentials

- No credentials stored here
- OTP exchange for user secrets during sessions
- Platform credentials (Telegram token, Dropbox token, etc.) live in env vars on the server

## Server

_(to be filled in when deployed)_
