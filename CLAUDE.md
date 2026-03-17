# NanoClaw

Personal Claude assistant. See [README.md](README.md) for philosophy and setup. See [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) for architecture decisions.

## Quick Context

Single Node.js process that connects to Telegram (primary channel), routes messages to Claude Agent SDK running in containers. Each group has isolated filesystem and memory. Deployed on a DigitalOcean VPS (Ubuntu 24.04, amd64).

## Key Files

| File | Purpose |
|------|---------|
| `src/index.ts` | Orchestrator: state, message loop, agent invocation |
| `src/channels/whatsapp.ts` | WhatsApp connection, auth, send/receive |
| `src/channels/telegram.ts` | Telegram bot connection (primary channel) |
| `src/ipc.ts` | IPC watcher and task processing |
| `src/router.ts` | Message formatting and outbound routing |
| `src/config.ts` | Trigger pattern, paths, intervals |
| `src/container-runner.ts` | Spawns agent containers with mounts |
| `src/task-scheduler.ts` | Runs scheduled tasks |
| `src/db.ts` | SQLite operations |
| `groups/{name}/CLAUDE.md` | Per-group memory (isolated) |
| `container/skills/agent-browser.md` | Browser automation tool (available to all agents via Bash) |

## Skills

| Skill | When to Use |
|-------|-------------|
| `/setup` | First-time installation, authentication, service configuration |
| `/customize` | Adding channels, integrations, changing behavior |
| `/debug` | Container issues, logs, troubleshooting |
| `/update` | Pull upstream NanoClaw changes, merge with customizations, run migrations |
| `/qodo-pr-resolver` | Fetch and fix Qodo PR review issues interactively or in batch |
| `/get-qodo-rules` | Load org- and repo-level coding rules from Qodo before code tasks |

## VPS Deployment

- **Auto-deploy**: Push to `main` triggers GitHub Actions (`.github/workflows/deploy.yml`)
- Container image only rebuilds when `container/` files change
- Deploy failures send a Telegram alert
- `.env` lives on server only — never in repo or CI
- SSH as `deploy` user; service runs via system-level systemd
- Daily SQLite backups at 3am, log rotation at 4am

### Container Architecture

Container images must target **linux/amd64** (VPS architecture). Mac builds (arm64) won't work.

```bash
# Cross-compile for VPS from Mac
docker buildx build --platform linux/amd64 -t nanoclaw-agent:amd64 container/
docker save nanoclaw-agent:amd64 | gzip > /tmp/nanoclaw-agent.tar.gz
# Upload: scp /tmp/nanoclaw-agent.tar.gz deploy@<VPS>:/tmp/
# On VPS: docker load < /tmp/nanoclaw-agent.tar.gz && docker tag nanoclaw-agent:amd64 nanoclaw-agent:latest
```

### VPS Commands (via SSH)

```bash
sudo systemctl restart nanoclaw   # restart service
journalctl -u nanoclaw -f         # tail logs
```

## Development

Run commands directly—don't tell the user to run them.

```bash
npm run dev          # Run with hot reload
npm run build        # Compile TypeScript
./container/build.sh # Rebuild agent container (local arch only)
```

Service management:
```bash
# macOS (launchd)
launchctl load ~/Library/LaunchAgents/com.nanoclaw.plist
launchctl unload ~/Library/LaunchAgents/com.nanoclaw.plist
launchctl kickstart -k gui/$(id -u)/com.nanoclaw  # restart

# Linux/VPS (systemd)
sudo systemctl start nanoclaw
sudo systemctl stop nanoclaw
sudo systemctl restart nanoclaw
```

## Container Build Cache

The container buildkit caches the build context aggressively. `--no-cache` alone does NOT invalidate COPY steps — the builder's volume retains stale files. To force a truly clean rebuild, prune the builder then re-run `./container/build.sh`.

## Integrations

| Integration | How Configured | Main-only |
|-------------|---------------|-----------|
| Notion | `NOTION_API_KEY` env var → MCP server in agent-runner | No |
| Google (Docs, Drive, Sheets, Gmail, Calendar, Slides, Tasks, People, Forms, Chat, YouTube, Meet) | `~/.google-mcp/` mounted into container → MCP server | Yes |
| GitHub | `GH_TOKEN` env var → passed to SDK env | No |
