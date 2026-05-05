This folder contains the source for a Skilled Agent originally built for the Valet runtime. Changes should follow the Skilled Agent open standard.

## Setup

### Connectors

- **sentry-mcp**: The Sentry MCP server, authenticated via an internal integration token. The agent uses it to read issues, events, full stack traces, breadcrumbs, releases, and (only after explicit Slack confirmation) to comment on or resolve issues. Add it from the catalog at the org level so other Sentry-powered agents can share the token.
- **parallel-search-mcp**: Parallel's web search MCP. Keyless — no slot to configure. The agent uses it to search GitHub issues, pull requests, and library changelogs for known fixes matching the error.

### Channels

- **slack** (slack): The agent's per-agent Slack bot. Listens for @mentions and replies in-thread, and posts the webhook hypothesis to whichever channels the bot has been invited to. Slack writes use the auto-injected outbound Slack connector.
- **sentry-webhook** (sentry-webhook): Receives Sentry issue webhooks. After deploy, paste the agent's webhook URL into your Sentry internal integration so new issues and regressions fire the agent.

### Secrets

- **SENTRY_ACCESS_TOKEN** (on `sentry-mcp`): Sentry internal integration token. Required scopes: `event:read`, `issue:read`, `issue:write` (only used after explicit Slack confirmation for comment/resolve), `project:read`, `org:read`. Create it under *Settings → Developer Settings → New Internal Integration* in your Sentry org.
- **SENTRY_CLIENT_SECRET** (on `sentry-webhook`): The same internal integration's client secret, used to verify webhook signatures. Auto-managed by Valet, but the value must mirror what Sentry shows for the integration. If you rotate it in Sentry, update it here too.

### External Setup

1. Create a Sentry internal integration (*Settings → Developer Settings → New Internal Integration*) with the scopes listed above and webhooks enabled. Copy the **token** into `SENTRY_ACCESS_TOKEN` and the **client secret** into `SENTRY_CLIENT_SECRET`.
2. After deploy, copy the agent's `sentry-webhook` URL from the Valet dashboard and paste it into the same Sentry internal integration as the webhook URL. Subscribe to the `issue` resource (covers new issues and regressions).
3. Invite the agent's Slack bot to your on-call channel (or wherever you want hypotheses to land). The agent posts to every channel it's a member of — invite it to one focused channel, or several. If the bot has not been invited anywhere, the hypothesis is sent as a DM to the workspace install user with a one-line nudge to invite it somewhere.
4. Invite the bot to any additional channels where engineers should be able to @mention it for ad-hoc Sentry questions (e.g. an engineering channel for *"what broke in production?"* follow-ups).
5. To smoke-test without waiting for a real error, @mention the bot in Slack with *"any new errors in the last hour?"* — that exercises the Slack + Sentry path without the webhook.

## Customizing

- **Control where hypotheses post**: invite or remove the bot from channels in Slack — that's the only signal the agent uses. There is no channel name in the configuration.
- **Filter which environments fire**: configure the Sentry internal integration to only send webhooks for specific environments or projects. The agent honors whatever Sentry sends.
- **Tune the dedup window**: the agent skips reposting the same Sentry issue ID within 24h via `MEMORY.md`. Adjust the window in SOUL.md *Phase 5: Dedup*.
- **Change candidate cause cap**: the agent caps web-search candidate causes at 3. Adjust in SOUL.md if your team prefers more or fewer.
