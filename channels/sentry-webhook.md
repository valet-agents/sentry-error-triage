# Sentry Webhook Received

IMPORTANT: The Sentry JSON webhook payload appears directly below
these instructions in this same user message. It is NOT in a
file. Do NOT use Read, Bash, find, or any tool to locate it. Just
parse the JSON inline from the text you are reading.

A Sentry issue or regression fired. Run the **Webhook Workflow**
in SOUL.md (Phases 1-5): parse → enrich via `sentry-mcp` →
web-search via `parallel-search-mcp` → post hypothesis → record
dedup entry.

## What to extract from the payload

- `data.issue.id` (or `data.event.issue_id`) — Sentry issue ID,
  used as the dedup key
- `data.issue.web_url` (or construct from `project_slug` +
  `short_id`) — the Sentry issue URL, included on every post
- `data.issue.title` (or `data.event.title`) — error message
- `data.issue.culprit` — function/file the issue is attributed to
- `data.issue.level` — error / warning / fatal
- `data.event.environment`, `data.event.release` — env + release
- `data.event.project` (slug), `data.event.platform` — language
  / framework hint for the web search query
- `data.issue.isRegression` (or `data.event.is_regression`) —
  determines `*[REGRESSION]*` vs `*[NEW]*` tag
- `data.event.exception.values[*]` — exception type for the web
  search query

## Scope

Act only on **this specific issue**. Do not crawl recent issues,
do not summarize the whole project, do not post unrelated context.
One webhook → one hypothesis → one set of Slack posts (one per
invited channel).

## Steps

1. Parse the JSON payload inline from the user message (above).
2. Apply the **Dedup rule** below. If this issue ID was posted
   within the last 24h per `MEMORY.md`, **stop here — do nothing**.
3. Use `sentry-mcp` to fetch the full event: complete stack trace
   with the highlighted in-app frame, breadcrumbs, tags. Identify
   the **top in-app frame** (`file:line`) — skip third-party
   frames.
4. Build a web-search query from `<exception type> <message
   excerpt> <framework>` and call `parallel-search-mcp`. Distill
   at most 3 candidate causes, each with a one-line summary and
   source URL. Prioritize GitHub issues, GitHub PRs, and library
   changelogs. If nothing relevant, write
   `_No similar reports found._` instead.
5. Compose the message per SOUL **Phase 4: Post** format. Tag is
   `*[NEW]*` or `*[REGRESSION]*`. Include the Sentry issue URL on
   its own line. Cap message at 2,000 characters.
6. Resolve target Slack channels per SOUL **Where to post**:
   every channel the bot is a member of. If invited to none, DM
   the workspace install user instead with the hypothesis and a
   one-line invite hint.
7. Post once per resolved destination. Do not retry on failure —
   log the error and continue with the remaining destinations.
8. Append a dedup entry to `MEMORY.md`:
   `<issue-id> <ISO-timestamp>`. Trim entries older than 7 days.
9. Your turn ends after the posts. No follow-ups, no thread
   replies after the initial post.

## Dedup rule

Before posting, scan `MEMORY.md` for a line whose issue ID matches
this webhook's `data.issue.id`. If a matching entry is less than
24h old, **skip the post entirely**. Sentry will fire repeats for
the same issue when event volume crosses thresholds — the agent
posts once per issue per day, no more.
