---
name: web-router
description: >
  Primary entry point for any online task — web search, page fetch, scrape, real-time
  lookup, multi-source research, X / Twitter posts, site crawling, logged-in or
  anti-bot sites, browser automation, screenshots, and social-media reading
  (Xiaohongshu / Weibo / WeChat / 小红书 / 微博 / 公众号). Classifies the request once
  and dispatches to the cheapest tool that fits: built-in WebSearch / WebFetch / curl,
  mcp__web-search-fast (stealth-browser search and page content),
  mcp__grok-search (LLM-synthesized research, native X search, site mapping),
  playwright-cli (fresh-browser automation), or web-access (CDP to user's real
  logged-in browser). Use whenever the user asks to search, fetch, scrape, look up,
  browse, screenshot, or interact with anything online — including Chinese requests
  (搜索 / 查一下 / 抓取 / 看一下网页 / 打开 / 截图 / 登录).
metadata:
  type: router
  routes-to:
    - WebSearch
    - WebFetch
    - curl
    - mcp__web-search-fast
    - mcp__grok-search
    - playwright-cli
    - web-access
---

# web-router — unified entry point for network tasks

## Purpose

A **dispatcher**, not an executor. It tells you which underlying tool to call for a given online request; you still invoke the chosen tool directly.

**Non-goals** — stop and defer when the request is actually:

- A document file (PDF / DOCX / PPTX / XLSX / image) → `mineru-document-extractor`
- A PubMed / medical-literature query → `pubmed-skill`
- A request to *write* a userscript / page UI → `frontend-dedup`
- No online component at all

## Step 1 — Classify

Answer in order. Stop at the first **yes**.

1. **Needs the user's logged-in session?** Named logged-in platform (Xiaohongshu inbox, my Weibo, my WeChat 公众号 后台, my company Notion, `.corp` / `.internal` domain), explicit "my account / my inbox / my orders", or anything that says "as me" → **`web-access` skill** (only tool with CDP to the real browser).
2. **Needs multi-step browser interaction?** Click, fill, screenshot a specific element, scrape after JS interaction, accept a cookie banner, record a flow → **`playwright-cli` skill** if anonymous; **`web-access`** if it also needs login.
3. **Otherwise: it's a read or search.** → Step 2.

## Step 2 — Dispatch table

| Scenario | Tool | Notes |
|---|---|---|
| Quick factual lookup ("when did X ship", "current price of Y") | `WebSearch` | Cheapest. Escalate to `mcp__web-search-fast__web_search` `depth=1` only on empty / stale results. |
| Real-time / very recent news (< 48h) | `mcp__web-search-fast__web_search` `engine="duckduckgo"` `depth=1` | Built-in `WebSearch` lags. |
| Read one known URL, static HTML / markdown / JSON | `WebFetch` | Cheapest URL reader. Use `curl` via `Bash` only for raw bytes / non-HTML. |
| Read one URL where `WebFetch` returned empty / JS shell / Cloudflare challenge | `mcp__web-search-fast__get_page_content` | Stealth browser, no login needed. |
| Multi-source research question with citations ("compare A and B", "summarize the state of X") | `mcp__grok-search__web_search` | Returns Grok's synthesized answer + `session_id`; follow up with `mcp__grok-search__get_sources` if raw sources needed. |
| Research focused on a single platform (Reddit / GitHub / Hacker News) | `mcp__grok-search__web_search` with `platform="Reddit"` etc. | Built-in platform filter. |
| X / Twitter content — posts, accounts, hashtags, date-bounded searches | `mcp__grok-search__x_search` | Native X API. Supports `x_handles`, `from_date`/`to_date`, `image_understanding`, `video_understanding`. Beats stealth-browser scraping. |
| Map a documentation site / discover all pages on a domain | `mcp__grok-search__web_map` | Graph traversal with `max_depth` and `instructions` filter. Start `max_depth=1–2`. |
| Read 3+ pages on one topic, need raw quotes | `mcp__web-search-fast__web_search` to find URLs, then loop `mcp__web-search-fast__get_page_content` | Verbatim text. |
| Read 3+ pages on one topic, need a digest | `mcp__grok-search__web_search` | LLM synthesizes for you. |
| Social media — public profile / public post (no login) | Try `mcp__web-search-fast__get_page_content` first; escalate to `web-access` on block. Twitter goes via `mcp__grok-search__x_search` instead. | Many public posts work without a session. |
| Social media — logged-in feed / inbox / DMs | `web-access` | Has the cookies. |
| Local browser history / bookmarks / open tabs | `web-access` | Only tool with this access. |
| Video frame / media file extraction from a page | `web-access` | Documented in its SKILL.md. |
| Document at a URL (PDF / DOCX / PPTX / image) | **Stop** — `mineru-document-extractor` | Out of scope. |
| PubMed / academic literature query | **Stop** — `pubmed-skill` | Out of scope. |

`mcp__grok-search__web_fetch` exists but is functionally equivalent to built-in `WebFetch` (static HTML, no JS). Prefer `WebFetch`; use `mcp__web-search-fast__get_page_content` when JS rendering is needed.

`mcp__grok-search` also exposes `plan_*`, `switch_model`, `toggle_builtin_tools`, `get_config_info` — these are internal / admin helpers, not for routine routing.

## Step 3 — Escalation (cheap → heavy)

If the chosen tool fails, escalate by one tier — do not retry the same tool blindly.

- `403` / `429` / Cloudflare / empty body from `WebFetch` → `mcp__web-search-fast__get_page_content`.
- Stale or empty `WebSearch` → `mcp__web-search-fast__web_search` `depth=1`.
- Stealth-browser MCP also blocked, or login wall → `web-access`.
- Page renders only after click / scroll / form-fill → `playwright-cli` (anon) or `web-access` (logged-in).
- `WebFetch` returns a JS shell (empty `<body>`) → `mcp__web-search-fast__get_page_content`.

Skip tiers when Step 1 already gave a strong signal (e.g. "check my Xiaohongshu inbox" → jump straight to `web-access`).

## Cost / latency

- `web-access` opens **real browser tabs visible to the user**. Don't use for one-line lookups.
- `mcp__grok-search__*` makes paid LLM calls. Don't use for a single factual question that `WebSearch` answers in 200ms.
- `playwright-cli` spawns a fresh browser process — fine for a single automation, wasteful for "fetch this URL".
- `mcp__web-search-fast__web_search` `depth=2` fetches full content for every result — only when you actually need the text; otherwise `depth=1`.

## Examples

| Request | Decision |
|---|---|
| `"latest Claude 4.7 release notes"` | `mcp__web-search-fast__web_search` `depth=1` |
| `"compare R2 vs S3 pricing for cold storage"` | `mcp__grok-search__web_search` |
| `"what's @elonmusk been posting about AI this week"` | `mcp__grok-search__x_search` with `x_handles="elonmusk"`, recent `from_date` |
| `"list all pages under docs.anthropic.com/claude-code"` | `mcp__grok-search__web_map` `max_depth=2` |
| `"what's in my Xiaohongshu inbox"` | `web-access` |
| `"screenshot the hero section of example.com"` | `playwright-cli` |
| `"read https://example.com/docs/api"` | `WebFetch`, fall back to `mcp__web-search-fast__get_page_content` if JS-rendered |
| `"调研一下 2026 年向量数据库的选型"` | `mcp__grok-search__web_search` |
| `"提取这个 PDF 里的表格"` | **Stop** — `mineru-document-extractor` |

## Execution Rules

- Execute independent search requests in parallel; sequential execution applies only when dependencies exist.
- Evaluate search results for quality: analyze relevance, source credibility, cross-source consistency, and completeness. Conduct supplementary searches if gaps exist.
- When calling `mcp__grok-search__web_search`, append the following to the end of queries: `Please list all reference sources at the end under a "Sources" heading, formatted as - [title](url)`