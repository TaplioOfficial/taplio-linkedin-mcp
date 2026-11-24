# Tool reference

Full reference for the 37 tools exposed by the Taplio LinkedIn MCP Server. Every tool acts only on the
authenticated user's own Taplio and LinkedIn account. Endpoint: `https://mcp.taplio.com/` (Streamable HTTP,
OAuth). Full input and output schemas are returned by the server (`tools/list`).

Contents:

- [Classification](#classification)
- [Quick index](#quick-index)
- [Tools](#tools)
- [Status handling](#status-handling)
- [Errors](#errors)
- [Rate limits and quotas](#rate-limits-and-quotas)
- [Plans](#plans)
- [Examples](#examples)

---

## Classification

Each tool falls into one of three classes:

- **Read**: retrieves data and changes nothing.
- **Write**: changes the user's private Taplio workspace only (drafts, media, an action still waiting in the queue).
  Nothing becomes visible on LinkedIn.
- **Sensitive write**: puts something on LinkedIn under the user's name, now or later, or deletes data
  permanently. Show the user what will happen and get their approval before every call.

The MCP annotations returned in `tools/list` follow the same split: read tools carry `readOnlyHint: true`, and the
tools that act on LinkedIn carry `openWorldHint: true`.

---

## Quick index

| Tool                        | Toolset                | Class           | Plan    |
| --------------------------- | ---------------------- | --------------- | ------- |
| `get_me`                    | Account and analytics  | Read            | Any     |
| `list_posts`                | Account and analytics  | Read            | Any     |
| `get_post`                  | Account and analytics  | Read            | Any     |
| `get_analytics_overview`    | Account and analytics  | Read            | Any     |
| `get_post_analytics`        | Account and analytics  | Read            | Any     |
| `list_drafts`               | Drafts and media       | Read            | Any     |
| `get_draft`                 | Drafts and media       | Read            | Any     |
| `create_draft`              | Drafts and media       | Write           | Any     |
| `update_draft`              | Drafts and media       | Write           | Any     |
| `delete_draft`              | Drafts and media       | Sensitive write | Any     |
| `list_media`                | Drafts and media       | Read            | Any     |
| `create_media_upload_url`   | Drafts and media       | Write           | Any     |
| `attach_media_from_url`     | Drafts and media       | Write           | Any     |
| `pick_media`                | Drafts and media       | Write           | Any     |
| `upload_media`              | Drafts and media       | Write           | Any     |
| `schedule_draft`            | Schedule and publish   | Sensitive write | Starter |
| `unschedule`                | Schedule and publish   | Write           | Starter |
| `publish_draft`             | Schedule and publish   | Sensitive write | Starter |
| `search_inspiration`        | Inspiration            | Read            | Any     |
| `find_posts_to_comment`     | Engage                 | Read            | Growth  |
| `list_comment_feeds`        | Engage                 | Read            | Growth  |
| `find_comments_to_reply`    | Engage                 | Read            | Growth  |
| `list_comment_drafts`       | Engage                 | Read            | Growth  |
| `get_comment_draft`         | Engage                 | Read            | Growth  |
| `create_comment_draft`      | Engage                 | Write           | Growth  |
| `update_comment_draft`      | Engage                 | Write           | Growth  |
| `commit_comment_draft`      | Engage                 | Sensitive write | Growth  |
| `cancel_comment_draft`      | Engage                 | Write           | Growth  |
| `list_comments`             | Engage                 | Read            | Growth  |
| `get_comment`               | Engage                 | Read            | Growth  |
| `like_post`                 | Engage                 | Sensitive write | Growth  |
| `get_post_like`             | Engage                 | Read            | Growth  |
| `cancel_like`               | Engage                 | Write           | Growth  |
| `send_connection_request`   | Network                | Sensitive write | Pro     |
| `list_connection_requests`  | Network                | Read            | Pro     |
| `get_connection_request`    | Network                | Read            | Pro     |
| `cancel_connection_request` | Network                | Write           | Pro     |

19 read, 12 write, 6 sensitive write. "Any" means the tool works on every account, including the free calls of an
account without a plan (see [Plans](#plans)).

---

## Tools

### Account and analytics

- **`get_me`** (Read). The user's display name and LinkedIn handle, content preferences (industry, role, language,
  audience, topics, keywords) and today's LinkedIn stats (followers, connections, profile views). Call it first.
- **`list_posts`** (Read). The user's posts, newest first: posts created in Taplio (`scheduled`, `sending`, `sent`)
  and posts published directly on LinkedIn (always `sent`, last 90 days, synced daily). Filters: `status`, `from`,
  `to`. Includes each post's format and media, not its metrics.
- **`get_post`** (Read). One post, by Taplio id or by the LinkedIn urn `list_posts` returns.
- **`get_analytics_overview`** (Read). Account metrics per day: followers, connections, profile views,
  impressions, likes, comments, shares. Defaults to today; range capped at 90 days.
- **`get_post_analytics`** (Read). Per-post impressions, likes, comments and shares, with each post's format.
  Defaults to the last 7 days; range capped at 90 days.

### Drafts and media

- **`list_drafts`** (Read). Unpublished drafts. Drafts are the only editable stage of a post.
- **`get_draft`** (Read). One draft by id.
- **`create_draft`** (Write). Creates an unpublished draft, 1 to 3000 characters, optionally with media
  (`media_ids`) and a title for an attached PDF. Each call creates a new draft.
- **`update_draft`** (Write). Replaces the text of a draft and can add media. Media already attached is kept.
  Scheduled and sent posts cannot be edited.
- **`delete_draft`** (Sensitive write). Deletes a draft permanently. It cannot be restored.
- **`list_media`** (Read). Media already in the user's Taplio library (id, type, url, filename), newest first,
  optionally filtered by filename.
- **`create_media_upload_url`** (Write). Returns a `media_id` and a one-time upload URL for an image, video, GIF or
  PDF. The client uploads the bytes to that URL, then passes the `media_id` to `create_draft` or `update_draft`.
- **`attach_media_from_url`** (Write). Downloads a public https file (jpeg, png, webp, gif, mp4, mov, webm, pdf) and
  attaches it to a draft. If the URL is not reachable, the draft stays text-only.
- **`pick_media`** (Write). Opens an interactive panel listing the user's media library; the media the user clicks
  is attached to the draft. Needs a client that supports MCP Apps.
- **`upload_media`** (Write). Opens an interactive panel where the user drops or chooses a new file, which is
  uploaded and attached to the draft. Needs a client that supports MCP Apps. In other clients, use
  `create_media_upload_url` or `attach_media_from_url`.

### Schedule and publish

- **`schedule_draft`** (Sensitive write). Schedules a draft to be published on LinkedIn under the user's name at
  `scheduled_for` (ISO 8601, at least 2 minutes ahead). Reversible with `unschedule` until it goes out.
- **`unschedule`** (Write). Moves a scheduled post back to drafts.
- **`publish_draft`** (Sensitive write). Publishes a draft on LinkedIn now. It cannot be undone.

### Inspiration

- **`search_inspiration`** (Read). Searches posts by other LinkedIn creators: free text, minimum likes or comments,
  author follower cap, age, length, language, or one author's posts. Counts toward a daily search allowance (see
  [Rate limits and quotas](#rate-limits-and-quotas)).

### Engage

- **`find_posts_to_comment`** (Read). Posts worth commenting on, from the user's own posts (`my_posts`), a curated
  feed (`curated_feed`) or a personalized selection (`for_you`). Each post carries the `post_urn` a comment needs.
- **`list_comment_feeds`** (Read). The curated feeds available to `find_posts_to_comment`.
- **`find_comments_to_reply`** (Read). Comments still worth a reply on 1 to 10 posts. Comments the user wrote,
  already answered or hid are left out. Reading a single post by its LinkedIn URL needs the Pro plan.
- **`list_comment_drafts`** (Read). Comment drafts, newest first, whatever their status. This is also the queue and
  the history.
- **`get_comment_draft`** (Read). One comment draft. This is how to confirm what a commit did.
- **`create_comment_draft`** (Write). Drafts a comment on a post, or a reply to a comment (`comment_id`), 1 to 1250
  characters. Nothing reaches LinkedIn. One comment per post and one reply per comment: a second one is refused.
- **`update_comment_draft`** (Write). Rewrites a comment draft that is still in status `draft`.
- **`commit_comment_draft`** (Sensitive write). Queues the comment to be posted on LinkedIn under the user's name.
  It is not live when the call returns: see [Status handling](#status-handling). Do not retry a commit.
- **`cancel_comment_draft`** (Write). Stops a committed comment before it is sent.
- **`list_comments`** (Read). Comments the user posted on other people's posts, through Taplio or directly on
  LinkedIn, with impressions, likes, replies and reactions (synced daily).
- **`get_comment`** (Read). One posted comment by its `comment_urn`, with the same stats.
- **`like_post`** (Sensitive write). Queues a like on a post or a comment, sent under the user's name. Asking
  again for a post already queued returns the same pending like.
- **`get_post_like`** (Read). Status of the latest like queued for a post or comment.
- **`cancel_like`** (Write). Stops a queued like before it is sent.

### Network

- **`send_connection_request`** (Sensitive write). Queues a connection request, without a note, to a LinkedIn
  member by public username. Asking again for someone already queued returns the same pending request.
- **`list_connection_requests`** (Read). Connection requests the account queued, newest first, up to 100.
- **`get_connection_request`** (Read). Status of the latest request for a username.
- **`cancel_connection_request`** (Write). Stops a queued connection request before it is sent.

---

## Status handling

**Posts.** A draft and a post are one item at different stages:
`draft` -> `scheduled` -> `sending` -> `sent`, or `draft` -> `sending` -> `sent` with `publish_draft`.
`unschedule` moves a `scheduled` post back to `draft`. A `sent` post cannot be changed.

**Comments, likes and connection requests are queued, not sent during the call.** Taplio paces them to keep the
account safe (posting window, office hours, hourly limits), so the response carries `scheduled_for`, the time the
action is due: minutes away, or hours later.

| Item               | Statuses                                                   | Follow up with           | Cancel with                 |
| ------------------ | ---------------------------------------------------------- | ------------------------ | --------------------------- |
| Comment draft      | `draft`, `pending`, then `posted`, `failed` or `cancelled` | `get_comment_draft`      | `cancel_comment_draft`      |
| Like               | `pending`, then `liked`, `failed` or `cancelled`           | `get_post_like`          | `cancel_like`               |
| Connection request | `pending`, then `sent`, `skipped`, `failed` or `cancelled` | `get_connection_request` | `cancel_connection_request` |

- `pending` means queued: nothing is on LinkedIn yet. Never report a pending action as done.
- A comment is live only once `get_comment_draft` shows `posted` with a `comment_urn`.
- `failed` and `cancelled` come with a `failure_reason`; a duplicate comment found when it is due is cancelled
  with code `COMMENT_ALREADY_POSTED`.
- A connection request reads `skipped` when the person is already a connection or already holds an invitation from
  the account. Nothing was sent.
- Once sent, an action is irreversible. Cancelling it after that is refused with `*_not_cancellable`.

---

## Errors

A failed call returns a tool result with `isError: true`.

- **Account and plan refusals** come back as one plain sentence with the link that fixes it: no plan or plan too
  low, free calls used up, payment failed, daily inspiration searches used up, LinkedIn not connected or
  disconnected. Relay it to the user.
- **Other errors** come back as JSON: `{ "code", "message", "request_id", "details" }`.

| HTTP | Codes                                                                                                                                                                                              | What to do                                             |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| 400  | `validation_failed`, `invalid_cursor`, `invalid_limit`                                                                                                                                             | Fix the input. `details` names the field.              |
| 401  | session expired or revoked                                                                                                                                                                         | Reconnect and approve access again.                    |
| 402  | `payment_failed`                                                                                                                                                                                   | Access is paused until the payment goes through.       |
| 403  | `insufficient_plan`                                                                                                                                                                                | The account needs a plan or a higher one.              |
| 404  | `post_not_found`, `draft_not_found`, `media_not_found`, `comment_draft_not_found`, `comment_not_found`, `connection_not_found`, `like_not_found`                                                    | Check the id or urn; do not retry as is.               |
| 409  | `draft_already_committed`, `draft_already_scheduled`, `comment_already_committed`, `comment_commit_blocked`, `comment_not_cancellable`, `connection_request_blocked`, `connection_not_cancellable`, `like_blocked`, `like_not_cancellable` | The item is already in that state. Do not retry.       |
| 422  | `draft_content_too_long`, `schedule_in_past`, `unsupported_media_type`, `media_conflict`, `analytics_invalid_range`, `url_content_unavailable`                                                     | Change the request (length, date, file type, range).   |
| 429  | `rate_limit_exceeded`, `inspiration_search_limit_exceeded`                                                                                                                                         | Wait: see [Rate limits and quotas](#rate-limits-and-quotas). |
| 5xx  | `internal_error`, `upstream_unavailable`                                                                                                                                                           | Retry later. Check the status first for queued actions. |

Before retrying any write, read the item back (`get_draft`, `get_comment_draft`, `get_post_like`,
`get_connection_request`) to avoid doing it twice. `like_post` and `send_connection_request` are safe to retry;
`publish_draft` and `commit_comment_draft` are not.

---

## Rate limits and quotas

- **Calls:** 600 per user per hour across all tools. Beyond that, `rate_limit_exceeded`; retry after an hour.
- **Free calls:** an account that never had a plan or trial gets 50 successful calls on the tools marked "Any".
- **Inspiration searches:** a daily allowance that resets at 00:00 UTC: 50 without a plan, on Starter and during
  the free trial, 150 on Growth, 300 on Pro. Each page counts.
- **Queued LinkedIn actions:** comments, likes and connection requests are paced by Taplio's send queue, so they
  can go out hours after the call.
- **Lists:** 25 items by default, 100 at most; paginate with `cursor` where offered.
- **Analytics:** ranges capped at 90 days.
- **Content:** posts 1 to 3000 characters, comments 1 to 1250. `scheduled_for` at least 2 minutes ahead.
- **`find_comments_to_reply`:** 1 to 10 posts per call.
- **`search_inspiration` defaults:** `min_likes` 50, `min_char_count` 500, `max_followers` 100000,
  `min_days_old` 1, `limit` 30. With `author`, `min_likes` and `min_char_count` default to 0 and `max_followers`
  to no cap.

---

## Plans

| Account                 | What it can use                                                                   |
| ----------------------- | --------------------------------------------------------------------------------- |
| No plan (never had one) | 50 free calls on the tools marked "Any"                                            |
| 7-day free trial        | Every tool                                                                         |
| Starter                 | "Any" tools, plus `schedule_draft`, `unschedule`, `publish_draft`                  |
| Growth                  | Starter tools, plus every Engage tool                                              |
| Pro                     | Every tool, including Network and `find_comments_to_reply` by post URL             |
| Plan ended              | Nothing until a plan is chosen again                                               |

Every tool needs a Taplio account; tools that read or act on LinkedIn need LinkedIn connected in Taplio.

---

## Examples

### Draft and schedule a post

```jsonc
// 1. Orient
get_me()

// 2. Create the draft
create_draft({ "content": "Here is what we learned shipping an MCP server..." })
// -> { "id": "drf_123", ... }

// 3. Schedule it (after the user approves the text and the time)
schedule_draft({ "id": "drf_123", "scheduled_for": "2026-07-14T07:30:00Z" })
```

### Weekly performance review

```jsonc
get_analytics_overview({ "from": "2026-07-01", "to": "2026-07-07" })
get_post_analytics({ "from": "2026-07-01", "to": "2026-07-07", "limit": 50 })
list_posts({ "status": "sent", "from": "2026-07-01", "to": "2026-07-07" })
```

### Find inspiration in a niche

```jsonc
search_inspiration({
  "query": "linkedin personal branding",
  "lang": "en",
  "min_likes": 200,
  "max_followers": 50000,
  "max_days_old": 30,
  "limit": 20
})
```

### Comment on a post

```jsonc
find_posts_to_comment({ "source": "for_you", "limit": 5 })
// -> [{ "post_urn": "urn:li:activity:123", ... }, ...]

create_comment_draft({ "post_urn": "urn:li:activity:123", "content": "Great point on..." })
// -> { "id": "cmd_456", "status": "draft", ... }

// After the user approves the text:
commit_comment_draft({ "id": "cmd_456" })
// -> { "status": "pending", "scheduled_for": "...", ... }

// Later, before telling the user it is live:
get_comment_draft({ "id": "cmd_456" })
// -> { "status": "posted", "comment_urn": "...", ... }
```
