# NanoClaw Setup Spec for Claude Code

## Goal
Deploy a forked NanoClaw instance on a Hetzner VPS (Ashburn, VA) connected via Telegram bot.
This serves as an AI operations assistant for Diamante Technical Coaching.

---

## Phase 1: Provision the VPS

### Provider
Hetzner Cloud — Ashburn, VA data center (US East region)

### Server Spec
- **Plan:** CPX21 (3 vCPU, 4GB RAM, 80GB NVMe SSD) — approximately $9.50/month
- **OS:** Ubuntu 24.04 LTS
- **Region:** `us-east` (Ashburn)

### What to Install
```
- Docker (latest stable, via official Docker apt repo — not snap)
- Node.js 20 LTS (via nvm)
- Git
- ufw (firewall)
- fail2ban
```

Note: nginx/certbot are NOT needed — Telegram uses outbound HTTPS polling, no inbound ports required beyond SSH.

### Firewall Rules (ufw)
```
- Allow 22/tcp (SSH) — restrict to my IP if possible
- Deny everything else
- Default deny incoming, allow outgoing
```

### Security Hardening
- Disable root SSH login
- Disable password SSH auth (key-only)
- Configure fail2ban with default SSH jail
- Auto-security updates enabled (unattended-upgrades)
- 1GB swap file as safety margin

---

## Phase 2: NanoClaw Installation

### Environment Variables Needed (prompt me for these, don't hardcode)
```
ANTHROPIC_API_KEY=        # Claude API key (requires API billing, NOT subscription)
TELEGRAM_BOT_TOKEN=       # From @BotFather
TELEGRAM_ONLY=true        # Skip WhatsApp entirely
ASSISTANT_NAME=Coach      # Trigger word
TZ=America/New_York       # For cron scheduling
NOTION_API_KEY=           # Notion internal integration token (optional)
```

### Docker Deployment
NanoClaw runs each agent session in an **isolated Docker container** — this is the security
model. Ensure:
- Docker socket is accessible to the NanoClaw process
- Container isolation is NOT disabled
- Each session gets its own filesystem, IPC namespace, process space
- Only explicitly mounted directories are accessible from within agent containers

### systemd Service
Create a systemd service so NanoClaw starts automatically on reboot:
```
- Service name: nanoclaw
- Restart policy: always
- Logs to journald
- Starts after Docker
```

### Health & Backup
- Cron job pinging Telegram bot API `getMe` endpoint, logging failures
- Daily SQLite backup of `store/messages.db`

---

## Phase 3: Telegram Bot Setup

1. Create bot via @BotFather: `/newbot`, set name "DTC Coach", username ending in "bot"
2. Copy bot token to `.env`
3. Optionally disable Group Privacy via BotFather (for group chat support)
4. Start a chat with the bot, send `/chatid` to get the chat ID
5. Register the main chat with `requiresTrigger: false`
6. Verify connection via test message

---

## Phase 4: Notion Integration

### Setup
- Create Notion internal integration at notion.so/my-integrations
- Share databases with the integration
- Add `NOTION_API_KEY` to `.env`
- The official `@notionhq/notion-mcp-server` runs inside agent containers

### Notion Database: Prospects CRM
Create manually with this schema:

| Property | Type |
|----------|------|
| Name | Title |
| Status | Select: New, Contacted, Qualified, Proposal, Won, Lost |
| Contact Date | Date |
| Notes | Rich Text |
| Source | Select: Referral, LinkedIn, Website, Event, Other |
| Email | Email |
| Next Follow-up | Date |

Store the database ID in `groups/main/CLAUDE.md` after creation.

---

## Phase 5: Agent Configuration

Agent persona and skills are defined in `groups/main/CLAUDE.md`:

**1. Morning Briefing (trigger: "good morning" or scheduled 8am ET weekdays)**
- Query Notion for open pipeline prospects and their last-contact date
- List any tasks/action items due today
- Format: short bullets, mobile-readable

**2. Prospect Capture (trigger: "log lead [name] [notes]")**
- Create new prospect entry in Notion CRM database
- Set status to "New", add today's date, save notes
- Confirm back with the entry details

**3. Quick Notion Search (trigger: "find [query]")**
- Search Notion workspace for matching pages/database entries
- Return top 3 results with titles

**4. Content Idea Queue (trigger: "content idea: [idea]")**
- Create entry in Content Ideas database
- Confirm saved

**5. Help (trigger: "help" or "what can you do")**
- Return a formatted list of available commands

---

## Phase 6: Validation Checklist

Before handing back, verify each item:

- [ ] VPS accessible via SSH with key auth only
- [ ] Root login disabled, fail2ban active
- [ ] Firewall: only port 22 open
- [ ] Docker running, container isolation working
- [ ] NanoClaw service running and enabled on boot
- [ ] Telegram bot connected and responding
- [ ] Send test message → agent responds
- [ ] Morning briefing trigger works
- [ ] Notion connection verified (log one test lead)
- [ ] systemd service survives `sudo reboot`
- [ ] Health check cron running
- [ ] SQLite backup cron running
- [ ] API spend limit set at console.anthropic.com

---

## Notes for Claude Code

- **Prompt for all secrets** — never generate placeholder API keys, always ask me
- **Commit incrementally** to the fork with descriptive commit messages
- **Document what you changed** from upstream NanoClaw in a `CHANGES.md` file at repo root
- If anything in NanoClaw's README contradicts this spec, flag it and ask me how to resolve

---

## Out of Scope (Do Not Implement Yet)

- LinkedIn integration
- Email/Gmail integration
- Calendar integration
- Newsletter automation
- Multi-agent swarms
- WhatsApp (using Telegram instead)

These will be added to the fork in future sessions.
