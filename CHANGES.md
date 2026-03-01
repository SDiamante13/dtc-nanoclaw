# Changes from Upstream NanoClaw

This fork is customized for Diamante Technical Coaching (DTC).

## Channel: Telegram (replaces WhatsApp)

- Applied `/add-telegram` skill — adds `src/channels/telegram.ts` using grammy
- Set `TELEGRAM_ONLY=true` to disable WhatsApp channel
- WhatsApp code remains in the codebase but is not loaded at runtime

**Files changed**: `src/index.ts`, `src/config.ts`, `src/routing.test.ts`
**Files added**: `src/channels/telegram.ts`, `src/channels/telegram.test.ts`

## Integration: Notion MCP Server

- Added `@notionhq/notion-mcp-server` to container dependencies
- Registered as MCP server in agent-runner (conditionally, when `NOTION_API_KEY` is set)
- Added `NOTION_API_KEY` to secret propagation chain (`readSecrets()`)

**Files changed**: `src/container-runner.ts`, `container/agent-runner/src/index.ts`, `container/agent-runner/package.json`

## Agent Persona

- Customized `groups/main/CLAUDE.md` with DTC Coach persona and business skills
- Skills: morning briefing, prospect capture, Notion search, content ideas, help

**Files changed**: `groups/main/CLAUDE.md`
