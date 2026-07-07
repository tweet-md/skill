---
name: tweet-md
description: "Gets X (Twitter) posts and threads as clean Markdown for LLMs via tweet.md. Use when the user wants to read, fetch, summarize, quote, or ingest an X post or thread (x.com/twitter.com link, tweet URL, or x.com→tweet.md rewrite) for an LLM, agent, or research. Also when they ask what's in a tweet/thread, to pull full conversation context, or to read replies in order."
version: 1.3.0
author: tweet.md
license: MIT
tags: [x, twitter, markdown, llm, agents, api, thread, conversion, rag]
platforms: [linux, macos, windows]
triggers:
  - read this tweet
  - read this X post
  - read this thread
  - tweet storm / reply chain
  - summarize this tweet
  - what does this post say
  - fetch x.com or twitter.com status
  - convert tweet to markdown
  - x.com to tweet.md
  - get thread for LLM or RAG
  - ingest tweet for agent
  - X Article / long-form post
  - user pastes x.com/status link
  - fetch profile / user profile
  - get bio / profile info
  - x.com or twitter.com profile URL
---

# tweet.md

X posts as clean Markdown for LLMs (`text/markdown`). No JSON.

**Base URL:** `https://tweet.md`  
**Docs:** https://tweet.md/i/docs · **Skill:** https://tweet.md/i/skill.md · **Credits:** https://tweet.md/i/topup · **API key:** https://tweet.md/i/dashboard

## When to invoke

Apply when user intent matches any **trigger** in the frontmatter above.

**Do not invoke** for: drafting or posting tweets, engagement metrics only, X developer app setup, or JSON API consumers (`format=json` is rejected).

## Choose a path

| Goal | Method |
|------|--------|
| Human/browser, one post | Replace `x.com` or `twitter.com` with `tweet.md` in the post URL |
| Human/browser, profile | Replace `x.com` or `twitter.com` with `tweet.md` in the profile URL |
| Programmatic, know handle + ID | `GET https://tweet.md/{handle}/status/{tweetId}` |
| Programmatic, only have full X URL | `GET https://tweet.md/i/api/convert?url={encoded_url}` |
| Programmatic, profile | `GET https://tweet.md/{handle}` or `GET https://tweet.md/i/api/profile?handle={handle}` |

**URL rewrite example:**

```
https://x.com/jack/status/20
→ https://tweet.md/jack/status/20
```

**Paid browser use:** After https://tweet.md/i/topup or https://tweet.md/i/login, the browser gets a session cookie — rewrite `x.com` → `tweet.md` with no `?apikey=` in the URL. Checkout sets the cookie on return from Stripe.

## Authentication

**Browser (human):** session cookie after https://tweet.md/i/topup or https://tweet.md/i/login — preferred for URL rewrites in the same browser.

**Programmatic:** `twmd_key_...` via `Authorization: Bearer ...` or `?apikey=`.

Free (no key): 5 single-post requests per IP per calendar month — `thread=off` and `userinfo=off` only. With a key, omitted `thread` / `userinfo` default to `ancestors-20` / `author`.

## Pick `format`

| `format` | Use when |
|-----------|----------|
| `markdown` | Default; readable notes with stats, media, quotes, full X Articles when present |
| `obsidian` | Vault import (YAML frontmatter + headings) |

## Query parameters

### Post/thread params

| Param | Default | Values |
|-------|---------|--------|
| `format` | `markdown` | `markdown`, `obsidian` |
| `thread` | `off` (free); `ancestors-20` when paid and omitted | `off`, `ancestors[-N]`, `branch[-N]`, `all[-N]` (cap `2`–`500`, default N=20) |
| `userinfo` | `off` (free); `author` when paid and omitted | `off`, `author`, `all` |
| `apikey` | — (cookie in browser) | `twmd_key_...` or `Authorization: Bearer` |

**Precedence:** URL param > per-key dashboard defaults (https://tweet.md/i/dashboard) > system defaults.

- `thread=off` — single post only
- `thread=ancestors-11` — opened post + up to 10 ancestors (reply chain up to the original post; cap includes the requested post)
- `thread=branch-8` — ancestors first, then replies underneath the requested post (context-first cap)
- `thread=all-200` — opened post + 199 most recent other conversation posts, including other people's reply branches (rendered oldest-first)
- Aliases: `full` → `branch-20`, `conversation` → `all-20`, bare `N` → `branch-N`
- `userinfo=author` — profile URL, avatar, bio, and public metrics (+2 credits per unique author; default when omitted with a key)  
- `userinfo=all` — same author fields as `author` (+2 credits per unique author)

**Response headers (agents):** `X-Tweetmd-Posts-Returned`, `X-Tweetmd-Credits-Charged`, `X-Tweetmd-Thread-Cap`, `X-Tweetmd-Cap-Hit` — never appended to the Markdown body.

### Profile params

Append to any `https://tweet.md/{handle}` URL. Defaults: pinned post + 5 latest posts; replies and articles off.

| Param | Default | Values |
|-------|---------|--------|
| `pinnedpost` | `on` | `on`, `off` |
| `latest` | `5` | `off`, or `5`–`50` |
| `replies` | `off` | `off`, or `5`–`20` |
| `articles` | `off` | `off`, or `5`–`20` |

Profile fetching requires credits — no free tier.

## Credits

### Post/thread credits
- **1 credit** per post returned  
- **+2 credits** per unique author when `userinfo=author` or `userinfo=all`  

### Profile credits
- **2 credits** for base profile (name, bio, stats, pics)  
- **+1 credit** for pinned post (when enabled and present)  
- **+1 credit** per section post returned (latest, replies, articles)  

### Shared rules
- Cache hits do **not** discount price — caching is internal, not a user benefit  
- Free tier: `thread=off`, `userinfo=off`, 5/month per IP. **No profile access** on free tier.  
- On `402` or `429`, send the user to checkout (see below)

## Checkout (no API key yet)

Skip https://tweet.md/i/topup — link the user straight to Stripe:

- `https://tweet.md/i/checkout?pack=small` — $5 / 500 credits  
- `https://tweet.md/i/checkout?pack=medium` — $19 / 2,200 credits (best value)  
- `https://tweet.md/i/checkout?pack=big` — $49 / 6,000 credits  

After payment, tweet.md **emails the API key** (`twmd_key_…`) to the address entered at checkout. Existing keys can top up the same way or via https://tweet.md/i/topup.

## Examples

```bash
# Single post
curl -sS -H "Authorization: Bearer twmd_key_..." \
  "https://tweet.md/jack/status/20"

# Thread branch (ancestors + replies) → Obsidian
curl -sS -H "Authorization: Bearer twmd_key_..." \
  "https://tweet.md/jack/status/20?thread=branch-20&format=obsidian"

# From X URL only, with ancestor context
curl -sS -H "Authorization: Bearer twmd_key_..." \
  "https://tweet.md/i/api/convert?url=https%3A%2F%2Fx.com%2Fjack%2Fstatus%2F20&thread=ancestors-20"

# Whole conversation, up to 200 posts
curl -sS -H "Authorization: Bearer twmd_key_..." \
  "https://tweet.md/jack/status/20?thread=all-200"

# Profile (default: pinned + 5 latest)
curl -sS -H "Authorization: Bearer twmd_key_..." \
  "https://tweet.md/jack"

# Profile with custom sections
curl -sS -H "Authorization: Bearer twmd_key_..." \
  "https://tweet.md/jack?latest=10&replies=5&articles=5"
```

## X Articles

Long-form posts (timeline link → `https://x.com/i/article/{id}`) are fetched via the X API `article` field on the same status lookup. No extra query param. Output adds an **Article:** block with title, article URL, and full `plainText` after the post text. Use the **status URL**, not the article URL alone. Repo: `docs/articles.md`.

## Errors

Plain-text body (not JSON): `400` bad input · `401` bad key · `402` paid feature / no credits · `404` post unavailable · `429` free limit · `503` upstream/X API

## Agent rules

1. **Ask** for an existing `twmd_key_…` first. If the user has none (or hits `402`/`429`), link `https://tweet.md/i/checkout?pack=medium` (or `small` / `big`) — the key is emailed after payment.
2. **Preserve** source URL, author, post ID, and platform attribution in downstream output.  
3. **Prefer** `markdown` for agents and notes; `obsidian` when saving to a vault.  
4. **Use** `/i/api/convert` when the user supplies a full `x.com`/`twitter.com` link; use path form when you already have handle and ID.  
5. **For profiles:** replace `x.com/{handle}` with `tweet.md/{handle}` the same way you do for posts. Use `?latest=off&replies=off&articles=off` if the user only needs the bio and stats. Profiles require credits — no free tier.  
6. **Check** auth on free tier — defaults are `thread=ancestors-20` and `userinfo=author` with a key (URL params override per-key dashboard defaults, which override these); free tier allows `thread=off` and `userinfo=off` only.  
7. **Read** the `X-Tweetmd-*` response headers to see posts returned, credits charged, and whether the thread cap was hit — raise the `-N` cap and retry if `X-Tweetmd-Cap-Hit` is set and the user wants the full thread.  
8. On failure, surface the response body; for credit/limit errors, link checkout and note the API key is emailed after payment.
