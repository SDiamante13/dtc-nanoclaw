# NanoClaw Setup Spec for Claude Code

## Goal
Deploy a forked NanoClaw instance on a Hetzner VPS (Ashburn, VA) connected to a Google Voice
number via WhatsApp Web. This will serve as a personal AI assistant for Diamante Technical
Coaching business operations.

---

## Phase 1: Provision the VPS

### Provider
Hetzner Cloud — Ashburn, VA data center (US East region)

### Server Spec
- **Plan:** CPX21 (3 vCPU, 4GB RAM, 80GB NVMe SSD) — approximately €5.77/month
- **OS:** Ubuntu 24.04 LTS
- **Region:** `us-east` (Ashburn)

### What to Install
```
- Docker (latest stable, via official Docker apt repo — not snap)
- Docker Compose v2
- Node.js 20 LTS (via nvm)
- Git
- ufw (firewall)
- fail2ban
- nginx (for reverse proxy)
- certbot (for TLS via Let's Encrypt, if domain is provided)
```

### Firewall Rules (ufw)
```
- Allow 22/tcp (SSH) — restrict to my IP if possible
- Allow 80/tcp (HTTP — for certbot challenge only)
- Allow 443/tcp (HTTPS — for reverse proxy)
- Deny everything else, including port 18789 (WhatsApp gateway must NOT be public)
- Default deny incoming, allow outgoing
```

### Security Hardening
- Disable root SSH login
- Disable password SSH auth (key-only)
- Configure fail2ban with default SSH jail
- Auto-security updates enabled (unattended-upgrades)

---

## Phase 2: NanoClaw Installation

### Environment Variables Needed (prompt me for these, don't hardcode)
```
ANTHROPIC_API_KEY=        # Claude API key
WHATSAPP_SESSION_NAME=    # e.g. "dtc-assistant"
AGENT_NAME=               # e.g. "Diamante" or whatever I want to call it
```

### Docker Deployment
NanoClaw runs each agent session in an **isolated Docker container** — this is the security
model. Ensure:
- Docker socket is accessible to the NanoClaw process
- Container isolation is NOT disabled
- Each session gets its own filesystem, IPC namespace, process space
- Only explicitly mounted directories are accessible from within agent containers

### nginx Reverse Proxy
- Proxy the WhatsApp web UI (typically port 3000 or 8080) behind nginx
- Require HTTP Basic Auth on the web UI (generate credentials, show me)
- Do NOT expose the WhatsApp gateway port (18789) through nginx or firewall
- Set up self-signed cert initially; note where to swap in Let's Encrypt if I add a domain

### systemd Service
Create a systemd service so NanoClaw starts automatically on reboot:
```
- Service name: nanoclaw
- Restart policy: always
- Logs to journald
- Starts after Docker
```

---

## Phase 3: WhatsApp Connection

After the service is running, walk me through:
1. How to access the QR code (URL or command)
2. How to scan it with the Google Voice WhatsApp number
3. How to verify the connection is live
4. What to do if the session drops (manual re-auth process)

Note: The Google Voice number will have WhatsApp already set up — I just need to scan
the QR code from the server.

---

## Phase 4: Initial Agent Configuration

Create a base system prompt / agent configuration file for a **business operations assistant**
with these capabilities wired up initially:

### Integrations to Configure (I'll provide credentials when prompted)
- **Notion** — for CRM, prospect tracking, content planning (Notion API key needed)
- **Anthropic Subscription** — already set above, use `claude-sonnet-4-5` as default model

### Initial Behaviors to Implement
These go in the agent's instruction/skill files:

**1. Morning Briefing (trigger: "good morning" or scheduled 8am ET)**
- Query Notion for open pipeline prospects and their last-contact date
- List any tasks/action items due today
- Remind of upcoming client sessions (if calendar integration added later)
- Format: short bullets, mobile-readable

**2. Prospect Capture (trigger: "log lead [name] [notes]")**
- Create new prospect entry in Notion CRM database
- Set status to "New", add today's date, save notes
- Confirm back with the entry details

**3. Quick Notion Search (trigger: "find [query]")**
- Search Notion workspace for matching pages/database entries
- Return top 3 results with titles and links

**4. Content Idea Queue (trigger: "content idea: [idea]")**
- Append idea to a "Content Ideas" page in Notion
- Confirm saved

**5. Help (trigger: "help" or "what can you do")**
- Return a formatted list of available commands

### Notion Database IDs Needed
Prompt me for these after Notion is connected:
- Prospect/CRM database ID
- Content ideas page ID
- Action items / tasks database ID (if exists)

---

## Phase 6: Validation Checklist

Before handing back, verify each item:

- [ ] VPS accessible via SSH with key auth
- [ ] Root login disabled, fail2ban active
- [ ] Docker running, container isolation working
- [ ] NanoClaw service running and enabled on boot
- [ ] Web UI accessible at https://[server-ip] (basic auth protected)
- [ ] Port 18789 NOT accessible from public internet (test with nmap or curl)
- [ ] WhatsApp QR code displayed and ready to scan
- [ ] Send test message → agent responds
- [ ] Morning briefing trigger works
- [ ] Notion connection verified (log one test lead)
- [ ] systemd service survives `sudo reboot`

---

## Notes for Claude Code

- **Ask before proceeding past Phase 2** — I need to verify VPS is provisioned and SSH works
- **Prompt for all secrets** — never generate placeholder API keys, always ask me
- **Show me the nginx basic auth credentials** — don't silently generate and store them
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
- Any browser automation

These will be added to the fork in future sessions.