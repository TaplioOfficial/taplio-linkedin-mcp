# Taplio LinkedIn MCP Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![MCP](https://img.shields.io/badge/protocol-MCP-blue.svg)](https://modelcontextprotocol.io)
[![Remote server](https://img.shields.io/badge/type-remote%20(HTTP)-brightgreen.svg)](#remote-taplio-mcp-server)

The Taplio LinkedIn MCP Server is a [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction)
server that connects AI tools directly to your [Taplio](https://taplio.com) account and, through it, to your
LinkedIn presence. It gives AI agents, assistants, and chat clients the ability to draft, schedule, and publish
LinkedIn posts, browse what you have already posted, research inspiration from other creators, and read your
analytics, all through natural language.

Point your MCP host at the Taplio server and you can ask things like:

- "Draft a LinkedIn post about our new MCP launch and schedule it for Tuesday 9am."
- "Show me my last 10 posts and which one got the most impressions."
- "Find 20 high-performing posts about `linkedin ghostwriting` from creators under 50k followers."
- "How are my followers and engagement trending over the last 30 days?"

## Use cases

- Turning raw ideas into publish-ready LinkedIn posts, on your own voice and topics.
- Automating draft creation, scheduling, and publishing from any MCP-capable client.
- Building AI-assisted content workflows and reporting on top of your real LinkedIn data.
- Extracting insights and analytics from your posting history without leaving your assistant.

---

## Contents

- [Remote Taplio MCP Server](#remote-taplio-mcp-server)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Claude Code](#claude-code)
  - [Claude Desktop / claude.ai connectors](#claude-desktop--claudeai-connectors)
  - [Manus](#manus)
  - [VS Code](#vs-code)
  - [Cursor](#cursor)
  - [Windsurf](#windsurf)
  - [Generic MCP config](#generic-mcp-config)
- [Authentication](#authentication)
- [Toolsets](#toolsets)
- [Tools](#tools)
  - [Profile](#profile)
  - [Drafts](#drafts)
  - [Publishing](#publishing)
  - [Posts](#posts)
  - [Analytics](#analytics)
  - [Inspiration](#inspiration)
- [Recommended workflow](#recommended-workflow)
- [Library and lifecycle model](#library-and-lifecycle-model)
- [Limits and safety](#limits-and-safety)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

## Remote Taplio MCP Server

Taplio is offered as a hosted, remote MCP server. There is nothing to install or run locally: your MCP host
connects to the Taplio endpoint over HTTP and authenticates with OAuth.

| Property        | Value                                                        |
| --------------- | ------------------------------------------------------------ |
| Transport       | Streamable HTTP (remote)                                     |
| Endpoint        | `https://mcp.taplio.com`                                     |
| Authentication  | OAuth 2.0 (authorize in the browser, no API key to paste)    |
| Scope           | The authenticated user's own Taplio + LinkedIn account       |
| Server hint     | Call [`get_me`](#get_me) first to load identity and settings |

> Because the server is remote and OAuth-based, you never copy a secret into a config file. Access is tied to
> the Taplio account you approve in the browser, and can be revoked from Taplio at any time.

---

## Prerequisites

1. An active [Taplio](https://taplio.com) account with LinkedIn connected.
2. An MCP-capable host (Claude Code, Claude Desktop, Manus, VS Code with Copilot, Cursor, Windsurf, or any client
   that speaks MCP over HTTP).
3. A browser available on first connect, to complete the OAuth authorization.

---

## Installation

### Claude Code

```bash
claude mcp add --transport http taplio https://mcp.taplio.com
```

Then run `/mcp` inside Claude Code and authorize `taplio` in the browser when prompted.

### Claude Desktop / claude.ai connectors

**One click:** open
[this link](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=Taplio&connectorUrl=https%3A%2F%2Fmcp.taplio.com)
to get the "Add custom connector" dialog already filled in with the Taplio name and URL, then click **Add**.

Or do it manually: in Claude Desktop or on claude.ai, open **Settings -> Connectors -> Add custom connector**, then
enter:

- **Name:** `Taplio`
- **URL:** `https://mcp.taplio.com`

Approve the OAuth screen. The Taplio tools then appear in the tool picker.

### Manus

Taplio is a native Manus plugin: one click, no server URL to paste.

1. Open the [Manus plugins directory](https://manus.im/app/plugins#connector_b0b849c6-2cab-4a24-b300-4df5de49a4de).
2. Find **Taplio** and click **Connect**.
3. Sign in to your Taplio account and approve access.

The Taplio tools are then available in your Manus tasks.

### VS Code

Add to your user or workspace MCP configuration (`.vscode/mcp.json`):

```json
{
  "servers": {
    "taplio": {
      "type": "http",
      "url": "https://mcp.taplio.com"
    }
  }
}
```

### Cursor

In `~/.cursor/mcp.json` (or **Settings -> MCP -> Add**):

```json
{
  "mcpServers": {
    "taplio": {
      "url": "https://mcp.taplio.com"
    }
  }
}
```

### Windsurf

In `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "taplio": {
      "serverUrl": "https://mcp.taplio.com"
    }
  }
}
```

### Generic MCP config

Any client that supports remote MCP servers over HTTP can use:

```json
{
  "mcpServers": {
    "taplio": {
      "type": "http",
      "url": "https://mcp.taplio.com"
    }
  }
}
```

---

## Authentication

The Taplio MCP Server uses OAuth 2.0. On first use your client opens a browser window where you log in to Taplio
and approve access. The server then acts on behalf of that single account. No API keys, tokens, or LinkedIn
credentials are stored in your client config. Revoke access at any time from your Taplio account settings.

---

## Toolsets

The tools are grouped into functional toolsets. All of them operate on the authenticated user's own account.

| Toolset         | Description                                                                 |
| --------------- | --------------------------------------------------------------------------- |
| **Profile**     | Identity, content preferences, and current LinkedIn stats.                  |
| **Drafts**      | Create, read, list, edit, and delete unpublished post drafts.               |
| **Publishing**  | Publish a draft now, schedule it, or unschedule it back to drafts.          |
| **Posts**       | Browse scheduled, sending, and sent posts.                                  |
| **Analytics**   | Account-level overview and per-post performance metrics.                    |
| **Inspiration** | Search other creators' LinkedIn posts to model content on what works.       |

---

## Tools

### Profile

#### `get_me`

Read this first to orient. Returns the user's identity and content preferences so you can tailor output to them
and recognize their own posts: display name and LinkedIn `@handle` (a user may go by several aliases, so check
both), plus `ai_settings` (industry, role, language, target audience, topics, keywords, description) and today's
LinkedIn stats (followers, connections, profile views).

_No parameters._

---

### Drafts

#### `create_draft`

Create a new LinkedIn post draft (unpublished). Publish it later with `schedule_draft` or `publish_draft`.

| Parameter | Type   | Required | Description                                            |
| --------- | ------ | -------- | ------------------------------------------------------ |
| `content` | string | yes      | The text body of the draft. 1 to 3000 characters.      |

#### `get_draft`

Fetch a single LinkedIn post draft by id.

| Parameter | Type   | Required | Description                          |
| --------- | ------ | -------- | ------------------------------------ |
| `id`      | string | yes      | The id of the draft (from `list_drafts`). |

#### `list_drafts`

List the user's unpublished LinkedIn post drafts.

| Parameter | Type    | Required | Description                                             |
| --------- | ------- | -------- | ------------------------------------------------------- |
| `limit`   | integer | no       | Max drafts to return. Defaults to 25, capped at 100.    |
| `cursor`  | string  | no       | Pagination token from a previous response.              |

#### `update_draft`

Update the content of an existing draft. Only drafts can be edited: published or scheduled posts cannot.

| Parameter | Type   | Required | Description                                                    |
| --------- | ------ | -------- | -------------------------------------------------------------- |
| `id`      | string | yes      | The id of the draft to update (from `list_drafts`).            |
| `content` | string | yes      | The new full text body (replaces existing). 1 to 3000 chars.   |

#### `delete_draft`

Permanently delete a LinkedIn post draft.

| Parameter | Type   | Required | Description                                          |
| --------- | ------ | -------- | ---------------------------------------------------- |
| `id`      | string | yes      | The id of the draft to permanently delete.           |

---

### Publishing

#### `publish_draft`

Publish a draft to LinkedIn immediately. This goes live now and cannot be undone.

| Parameter | Type   | Required | Description                                   |
| --------- | ------ | -------- | --------------------------------------------- |
| `id`      | string | yes      | The id of the draft to publish immediately.   |

#### `schedule_draft`

Schedule a draft to be published automatically at a future time.

| Parameter       | Type   | Required | Description                                                                     |
| --------------- | ------ | -------- | ------------------------------------------------------------------------------- |
| `id`            | string | yes      | The id of the draft to schedule (from `list_drafts`).                           |
| `scheduled_for` | string | yes      | ISO 8601 datetime (e.g. `2026-06-10T14:30:00Z`). At least 2 minutes in the future. |

#### `unschedule`

Unschedule a scheduled post, returning it to drafts so you can edit, reschedule, or delete it.

| Parameter | Type   | Required | Description                                          |
| --------- | ------ | -------- | ---------------------------------------------------- |
| `id`      | string | yes      | The id of the scheduled post (from `list_posts`).    |

---

### Posts

#### `list_posts`

List the user's LinkedIn posts (status `scheduled`, `sending`, or `sent`), most recent first. A draft becomes a
post once it is scheduled or published.

| Parameter | Type    | Required | Description                                                              |
| --------- | ------- | -------- | ----------------------------------------------------------------------- |
| `status`  | enum    | no       | Filter by `scheduled`, `sending`, or `sent`. Omit for all three.        |
| `from`    | string  | no       | Only posts on or after this date (YYYY-MM-DD or ISO 8601, UTC).         |
| `to`      | string  | no       | Only posts on or before this date (YYYY-MM-DD or ISO 8601, UTC).        |
| `limit`   | integer | no       | Max posts to return. Defaults to 25, capped at 100.                      |
| `cursor`  | string  | no       | Pagination token from a previous response.                              |

#### `get_post`

Fetch a single LinkedIn post by id. A post has status `scheduled`, `sending`, or `sent`; drafts are separate
(use `get_draft`).

| Parameter | Type   | Required | Description                                |
| --------- | ------ | -------- | ------------------------------------------ |
| `id`      | string | yes      | The id of the post (from `list_posts`).    |

---

### Analytics

#### `get_analytics_overview`

Get the user's LinkedIn analytics overview (followers, impressions, engagement). Defaults to today when no date
range is given.

| Parameter     | Type   | Required | Description                                                                                                    |
| ------------- | ------ | -------- | -------------------------------------------------------------------------------------------------------------- |
| `from`        | string | no       | Start date (YYYY-MM-DD, UTC). Defaults to today; range capped at 90 days.                                       |
| `to`          | string | no       | End date (YYYY-MM-DD, UTC). Defaults to today; range capped at 90 days.                                         |
| `metrics`     | string | no       | Comma-separated subset: `followers, connections, profile_views, impressions, likes, comments, shares`. Omit for all. |
| `granularity` | enum   | no       | Time-bucket size; only `day` is supported (default).                                                           |

#### `get_post_analytics`

List the user's per-post analytics (impressions, likes, comments, shares), defaulting to the last 7 days.

| Parameter | Type    | Required | Description                                                            |
| --------- | ------- | -------- | --------------------------------------------------------------------- |
| `from`    | string  | no       | Start date (YYYY-MM-DD, UTC). Defaults to 7 days ago; range capped at 90 days. |
| `to`      | string  | no       | End date (YYYY-MM-DD, UTC). Defaults to today; range capped at 90 days. |
| `limit`   | integer | no       | Max posts to return. Defaults to 25, capped at 100.                    |
| `cursor`  | string  | no       | Pagination token from a previous response.                            |

---

### Inspiration

#### `search_inspiration`

Search LinkedIn posts from other creators to use as inspiration.

| Parameter        | Type    | Required | Description                                                                       |
| ---------------- | ------- | -------- | --------------------------------------------------------------------------------- |
| `query`          | string  | no       | Free-text search over post content. Omit to browse top posts.                     |
| `lang`           | string  | no       | Filter by ISO 639-1 language code (e.g. `en`, `fr`).                               |
| `min_likes`      | integer | no       | Only posts with at least this many likes. 0-1000; defaults to 50.                 |
| `min_comments`   | integer | no       | Only posts with at least this many comments. 0-1000; defaults to 0.               |
| `min_char_count` | integer | no       | Only posts with at least this many characters. 0-10000; defaults to 500.          |
| `max_followers`  | integer | no       | Only posts whose author has at most this many followers. Defaults to 100000.      |
| `min_days_old`   | integer | no       | Only posts at least N days old (0-365); defaults to 1. Overrides `from`/`to`.      |
| `max_days_old`   | integer | no       | Only posts from the last N days (0-365). Overrides `from`/`to`.                    |
| `from`           | string  | no       | Only posts on or after this date. Ignored if `min`/`max_days_old` is set.          |
| `to`             | string  | no       | Only posts on or before this date. Ignored if `min`/`max_days_old` is set.         |
| `limit`          | integer | no       | Max posts to return. Minimum 1; defaults to 30.                                    |

---

## Recommended workflow

1. Call [`get_me`](#get_me) once at the start of a session to load the user's voice, topics, language, and stats.
2. Use [`search_inspiration`](#search_inspiration) to ground new content in what is currently working in the niche.
3. Draft with [`create_draft`](#create_draft), iterate with [`update_draft`](#update_draft).
4. Ship with [`schedule_draft`](#schedule_draft) (preferred) or [`publish_draft`](#publish_draft) (immediate).
5. Report with [`get_analytics_overview`](#get_analytics_overview) and
   [`get_post_analytics`](#get_post_analytics), and cross-reference with [`list_posts`](#list_posts).

---

## Library and lifecycle model

A **draft** and a **post** are the same item at different stages of its life:

```
create_draft ─┐
              ├─> [ DRAFT ] ──schedule_draft──> [ scheduled ] ──(auto)──> [ sending ] ──> [ sent ]
update_draft ─┘        │                              │
                       └──────publish_draft───────────┴──(immediate)──> [ sending ] ──> [ sent ]

                       [ scheduled ] ──unschedule──> [ DRAFT ]
```

- Drafts live in the **drafts** list (`list_drafts`) and are the only editable stage.
- Scheduling or publishing a draft turns it into a **post** and moves it to the **posts** list (`list_posts`).
- `unschedule` moves a scheduled post back to drafts; a `sent` post cannot be un-sent.

---

## Limits and safety

- **Post length:** 1 to 3000 characters (`create_draft`, `update_draft`).
- **Scheduling:** `scheduled_for` must be at least 2 minutes in the future, ISO 8601, UTC recommended.
- **Analytics range:** capped at 90 days per query; paginate with `cursor` for large result sets.
- **Irreversible actions:** `publish_draft` goes live immediately and `delete_draft` is permanent. Confirm intent
  before calling either.
- **Scope:** every tool acts only on the authenticated user's own Taplio and LinkedIn account.

---

## Troubleshooting

- **Tools do not appear:** confirm the server URL is `https://mcp.taplio.com` and that you completed the OAuth
  browser step. Re-run `/mcp` (Claude Code) or reconnect the connector.
- **`401` / auth errors:** your session may have expired or been revoked. Reconnect and re-authorize in the
  browser.
- **Cannot edit a post:** only drafts are editable. If it is already scheduled, call `unschedule` first; if it is
  `sent`, it can no longer be changed.
- **Empty analytics:** newly published posts take time to accumulate metrics; also check your date range against
  the 90-day cap.

---

## Contributing

This repository documents the Taplio LinkedIn MCP Server. See [CONTRIBUTING.md](./CONTRIBUTING.md) for how to
report issues with the documentation or suggest improvements. For product support, contact Taplio.

---

## License

This project is licensed under the terms of the MIT License. See [LICENSE](./LICENSE).
