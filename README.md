# Mellow Hub MCP

[Mellow Hub](https://www.mellow.world) is a hosted MCP server and REST API that
lets an AI agent (Claude, ChatGPT, Cursor and others), or a person, go from
trend to post: find the short videos taking off on TikTok, Instagram and
YouTube, then publish and schedule one post to ten social networks:
Instagram, TikTok, YouTube, X, LinkedIn, Threads, Bluesky, Pinterest, Facebook
and Telegram (channels, through Mellow's own bot). TikTok connects two ways, as
a personal account or as TikTok Business. Each network's rules are checked
before anything is sent.

This repository describes the hosted server. Its source code is not published
here, and there is nothing to build or run locally.

- Endpoint: `https://www.mellow.world/mcp` (Streamable HTTP), server version 1.9.0
- Setup guide: https://www.mellow.world/hub/guides/social-media-mcp
- Full reference: https://www.mellow.world/hub/docs (the same text as
  https://www.mellow.world/hub/llms.txt)
- Registry name: `world.mellow.www/hub` in the official MCP Registry
- Agent skill: https://github.com/wco1/mellow-hub-skill
- Made by Spody App, LLC. Contact: contact@mellow.world

## Connect

The endpoint speaks OAuth 2.1 with dynamic client registration and PKCE. A
client that supports OAuth discovery registers itself and shows the person a
consent screen. If a client cannot do OAuth, an API key that the owner creates
at https://www.mellow.world/hub/agents works as the bearer token on the same
endpoint.

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

In claude.ai or ChatGPT, add it as a custom connector with the same URL. Step
by step, per client and per network:
https://www.mellow.world/hub/guides/social-media-mcp

## What the server does

An OAuth connection is served 39 tools. A credential the owner grants the
`ai:generate` scope (a key made in Hub can carry it) also gets the Creative
Studio generation tools and `trends_breakdown`: 53 in all.

- **Start.** `whoami` says which channels this credential can reach, whether it
  publishes on its own or only prepares posts for a person to approve, and how
  many posts remain today. `list_channels` and `list_platforms` give the
  channel ids and every network's rules.
- **Check, then publish.** `validate_post` checks a post against every
  network's own rules (caption length, media count and type, required title or
  board) before anything is sent, and lists every problem at once.
  `preview_post` shows how it will read on each network. `create_post`
  publishes now or schedules; an idempotency key stops a retry from posting
  twice. Each network succeeds or fails on its own and returns its own link.
- **The rest of a post's life.** `get_post`, `list_posts`, `update_post`,
  `reschedule_post`, `retry_post` (sends again only to the networks that did
  not take it), `cancel_post` and `mark_post_finished` (for a post the person
  finished by hand in the network's app).
- **Media.** `request_upload_url` and `register_media` handle files and public
  URLs. `list_library_folders`, `list_library_assets` and `get_library_asset`
  read the photos and videos the person chose to share from their Mellow Hub
  Library with this connection, and `create_post` takes one by its id.
- **Trends.** `trends_search` finds the short videos taking off in a topic, one
  network a search (YouTube Shorts, TikTok or Instagram Reels): on TikTok and
  Instagram with each video's views against its creator's usual, on YouTube
  with YouTube's own figures. `trends_radar` keeps a standing search and checks
  it once a day. `trends_breakdown` says why one video worked and drafts the
  person's own version; it spends AI credits, so it needs `ai:generate`.
- **Results.** `post_results` (one post, per network), `account_overview` (the
  profile at a glance) and `get_metrics` (post-level figures the networks
  report).
- **Instagram comments and messages.** `list_autoreplies`, `create_autoreply`,
  `update_autoreply`, `delete_autoreply` and `catch_up_autoreplies` run
  keyword auto-replies: a word in a comment gets a public reply and a direct
  message. `list_comments`, `reply_to_comment`, `list_messages` and
  `reply_to_message` answer in the account's name, under an autopilot
  delegation only.
- **Creative Studio.** `studio_list`, `studio_job`, `studio_project`,
  `studio_save` and `studio_export` read, edit and export saved carousels and
  thumbnails; with `ai:generate`, the `studio_` generation tools make them.
- **Connecting and the journal.** `connect_channel` hands the person a link to
  connect the network themselves; an agent never receives a network password.
  `read_audit_log` reads this credential's own actions.

The full and current tool list, with arguments, is in the documentation.
Montage (clips cut into one vertical video with captions) and Stories (designed
Instagram stories) are part of the Hub web app, not MCP tools.

## Who is in control

Access is delegated by the owner, on a consent screen: the channels the agent
may touch, review or autopilot mode, a daily ceiling and an expiry. Every action
is written to a journal the owner can read at
https://www.mellow.world/hub/agents, and revoking a credential applies on the
next request. An agent credential cannot create another credential or approve
its own posts.

## Limits, stated plainly

- No team seats. Comments and messages are Instagram's only, through an
  account connected for auto-replies.
- TikTok blocks videos published through an app from an account whose region
  is Russia; for those, `isDraft` sends the video to the TikTok inbox to post
  from the phone.
- Mellow Hub is a paid service. Plans: https://www.mellow.world/hub/pricing

## More

- Browser tools (carousel maker, character and hashtag counters, image and
  video checkers): https://www.mellow.world/hub/tools
- Answers about limits and cross-posting: https://www.mellow.world/hub/answers
