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

**Read-only filesystem.** This agent cannot write to disk except via Dropbox.

- No local file writes (no logs, no temp files, no KB caching to disk)
- **Dropbox** is the only write surface — used for provisioned workspace outputs and reports
- All synthesized KB workspaces are written to Dropbox and a download link returned to the user
- Conversation state is in-memory only — no session persistence between messages unless authenticated

Implications for code:
- `KBWriter.write()` must target Dropbox, not local filesystem
- No `~/.switchboard/config.json` — config comes from env vars
- Audit session state is ephemeral; if the user drops off, they start over

## Credentials

- No credentials stored here
- OTP exchange for user secrets during sessions
- Platform credentials (Telegram token, Dropbox token, etc.) live in env vars on the server

## Server

_(to be filled in when deployed)_
