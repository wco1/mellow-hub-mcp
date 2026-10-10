# Mellow Hub MCP

Mellow Hub is a hosted MCP server and REST API that lets an AI agent (Claude,
ChatGPT, Cursor and others), or a person, publish and schedule posts to nine
social networks: Instagram, TikTok, YouTube, X, LinkedIn, Threads, Bluesky,
Pinterest and Facebook.

This repository describes the hosted server. Its source code is not published
here, and there is nothing to build or run locally.

- Endpoint: `https://www.mellow.world/mcp` (Streamable HTTP)
- Registry name: `world.mellow.www/hub` in the official MCP Registry
- Documentation: https://www.mellow.world/hub/docs
- Made by Spody App, LLC. Contact: contact@mellow.world

## Connect

The endpoint speaks OAuth 2.1 with dynamic client registration and PKCE. A
client that supports OAuth discovery registers itself and shows the person a
consent screen. If a client cannot do OAuth, an API key that the owner creates
in the cabinet works as the bearer token on the same endpoint.

Claude Code:

```sh
claude mcp add --transport http mellow https://www.mellow.world/mcp
```

Clients that read an `mcp.json`:

```json
{
  "mcpServers": {
    "mellow": {
      "type": "http",
      "url": "https://www.mellow.world/mcp"
    }
  }
}
```

In claude.ai or ChatGPT, add it as a custom connector with the same URL.

## What the server does

- `whoami` says which channels this credential can reach, whether it publishes
  on its own or only prepares posts for a person to approve, and how many posts
  remain today.
- `validate_post` checks a post against every network's own rules (caption
  length, media count and type, required title or board) before anything is
  sent, and lists every problem at once.
- `create_post` publishes now or schedules. Each network succeeds or fails on
  its own and returns its own link. Idempotency keys stop a retry from posting
  twice.
- `list_channels`, `list_platforms`, `preview_post`, `list_posts`, `get_post`,
  `post_results`, `cancel_post`, `reschedule_post` and `update_post` cover the
  rest of the life of a post; `request_upload_url` and `register_media` handle
  uploads; `get_metrics` returns post-level views.
- `connect_channel` hands the person to the Channels page; an agent never
  receives a network password.

The full and current tool list, with arguments, is in the documentation.

## Who is in control

Access is delegated by the owner, on a consent screen: the channels the agent
may touch, review or autopilot mode, a daily ceiling and an expiry. Every action
is written to a journal the owner can read, and revoking a credential applies
on the next request. An agent credential cannot create another credential or
approve its own posts.

## Limits, stated plainly

- No team seats, no inbox, and no comment or message management for users in
  general.
- TikTok blocks uploads from an account registered in Russia for everyone but
  the account's owner.
- Connecting accounts, drafts and validation are free. Publishing needs a
  subscription after a 7-day trial; plans start at $19 a month.

## More

- Free browser tools (carousel maker, character and hashtag counters, image and
  video checkers): https://www.mellow.world/hub/tools
- Answers about limits and cross-posting: https://www.mellow.world/hub/answers
- Pricing: https://www.mellow.world/hub/pricing
