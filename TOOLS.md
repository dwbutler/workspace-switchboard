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

**Read-only filesystem** with one exception: a write-only dropbox directory mounted into the container.

- The agent **cannot read from disk** (no session state, no cached data, no config files)
- The agent **can write to `/dropbox/`** (or whatever path is mounted) — but cannot read back what it wrote
- Write-only mount: outputs go in, the agent can't see them again
- Conversation state is in-memory only — ephemeral, isolated per session

**What goes into `/dropbox/`:**
- Provisioned workspace outputs (`.switchboard/` directories, `openclaw.json.template`)
- Audit reports
- Anything the operator or user needs to retrieve after the session

**Implications for code:**
- `KBWriter.write()` targets the dropbox mount path (env var: `DROPBOX_PATH`, default `/dropbox`)
- No `~/.switchboard/config.json` — all config via env vars
- Audit session state is ephemeral; if a user drops off, they start over
- The agent cannot exfiltrate or accumulate data — it literally can't read what it wrote

## Credentials

- No credentials stored here
- OTP exchange for user secrets during sessions
- Platform credentials (Telegram token, Dropbox token, etc.) live in env vars on the server

## Server

_(to be filled in when deployed)_
