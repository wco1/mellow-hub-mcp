# Mellow Hub MCP

Mellow Hub is a hosted MCP server and REST API that lets an AI agent (Claude,
ChatGPT, Cursor and others), or a person, publish and schedule posts to ten
social networks: Instagram, TikTok, YouTube, X, LinkedIn, Threads, Bluesky,
Pinterest, Facebook and Telegram (channels, through Mellow's own bot). It also
finds the short videos that took off in a topic on YouTube, TikTok and
Instagram.

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
- `trends_search` finds the videos that went furthest above their creator's
  usual views in a topic, one network a search, with each network's own public
  figures and a link to every video. `trends_breakdown` says why one worked and
  drafts the person's own version (it spends AI credits, so it needs a
  credential allowed to).
- `list_comments`, `reply_to_comment`, `list_messages`, `reply_to_message` and
  the auto-reply tools answer Instagram comments and direct messages in the
  account's name, under an autopilot delegation only.

The full and current tool list, with arguments, is in the documentation.

## Who is in control

Access is delegated by the owner, on a consent screen: the channels the agent
may touch, review or autopilot mode, a daily ceiling and an expiry. Every action
is written to a journal the owner can read, and revoking a credential applies
on the next request. An agent credential cannot create another credential or
approve its own posts.

## Limits, stated plainly

- No team seats. Comments and messages are Instagram's only, through an
  account connected for auto-replies.
- TikTok blocks uploads from an account registered in Russia for everyone but
  the account's owner.
- Hub is paid: plans start at $19 a month, after a 7-day trial that starts
  with a card. Cancel before it ends and nothing is charged.

## More

- Free browser tools (carousel maker, character and hashtag counters, image and
  video checkers): https://www.mellow.world/hub/tools
- Answers about limits and cross-posting: https://www.mellow.world/hub/answers
- Pricing: https://www.mellow.world/hub/pricing
