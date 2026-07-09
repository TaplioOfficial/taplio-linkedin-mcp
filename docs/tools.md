# Tool reference

Full reference for every tool exposed by the Taplio LinkedIn MCP Server. All tools operate on the authenticated
user's own Taplio and LinkedIn account. Endpoint: `https://mcp.taplio.com`.

Quick index:

| Tool                      | Toolset     | Mutating | One-line summary                                          |
| ------------------------- | ----------- | -------- | -------------------------------------------------------- |
| `get_me`                  | Profile     | no       | Identity, content preferences, and today's stats.        |
| `create_draft`            | Drafts      | yes      | Create an unpublished post draft.                        |
| `get_draft`               | Drafts      | no       | Fetch one draft by id.                                    |
| `list_drafts`             | Drafts      | no       | List unpublished drafts.                                 |
| `update_draft`            | Drafts      | yes      | Replace the content of a draft.                          |
| `delete_draft`            | Drafts      | yes      | Permanently delete a draft.                              |
| `publish_draft`           | Publishing  | yes      | Publish a draft immediately (irreversible).              |
| `schedule_draft`          | Publishing  | yes      | Schedule a draft for a future time.                      |
| `unschedule`              | Publishing  | yes      | Return a scheduled post to drafts.                       |
| `list_posts`              | Posts       | no       | List scheduled / sending / sent posts.                   |
| `get_post`                | Posts       | no       | Fetch one post by id.                                    |
| `get_analytics_overview`  | Analytics   | no       | Account-level metrics over a date range.                 |
| `get_post_analytics`      | Analytics   | no       | Per-post metrics over a date range.                      |
| `search_inspiration`      | Inspiration | no       | Search other creators' posts.                            |

---

## Examples

### Draft and schedule a post

```jsonc
// 1. Orient
get_me()

// 2. Create the draft
create_draft({ "content": "Here is what we learned shipping an MCP server..." })
// -> { "id": "drf_123", ... }

// 3. Schedule it
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

---

## Notes on defaults and caps

- `create_draft` / `update_draft` content: **1 to 3000** characters.
- `schedule_draft.scheduled_for`: ISO 8601, at least **2 minutes** in the future.
- Analytics date ranges: capped at **90 days**; `get_post_analytics` defaults to the **last 7 days**.
- List endpoints default to **25** items and cap at **100**; use `cursor` to paginate.
- `search_inspiration` defaults: `min_likes` 50, `min_char_count` 500, `max_followers` 100000, `min_days_old` 1,
  `limit` 30. Setting `min_days_old` or `max_days_old` overrides `from` / `to`.
