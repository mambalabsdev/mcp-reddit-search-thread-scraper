# Reddit Search and Thread Scraper MCP Server

[![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-reddit-search-thread-scraper)](https://www.npmjs.com/package/@mambalabsdev/mcp-reddit-search-thread-scraper) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-reddit-search-thread-scraper)](https://github.com/mambalabsdev/mcp-reddit-search-thread-scraper/blob/main/LICENSE)

An MCP server for the Mamba Labs Reddit Search and Thread Scraper actor on Apify. Search Reddit by keyword, read subreddits, or read threads with their comments, and get one flat row per post and comment.

> **Status, 12 November 2026**
>
> **Working:** keyword search inside subreddits, subreddit reads, and thread URLs with comments, through Arctic Shift, a free public archive of Reddit. Keyword search across all of Reddit works with your own Serper or Brave search key. Each post's current title, score, and comment count are read from Reddit's public embed page.
>
> **Changed:** Reddit closed logged out search on about 5 November 2026 and ends its RSS feeds on 13 November 2026, so the actor's version 0.1, which read those feeds, could no longer return posts. Actor version 0.2 and this server's version 1.1.0 do not use RSS. What to use instead: name a subreddit for every search (Arctic Shift needs one), or pass your own Serper or Brave key in `search_api_key` to search all of Reddit. A blocked run charges only the start fee.

## What's Inside

- [What it does](#what-it-does)
- [Routes and fallback](#routes-and-fallback)
- [Quick start](#quick-start)
- [Prerequisites](#prerequisites)
- [Example prompts](#example-prompts)
- [Tool](#tool)
- [Inputs](#inputs)
- [Pricing](#pricing)
- [FAQ](#faq)
- [Version notes](#version-notes)
- [Full actor documentation](#full-actor-documentation)
- [License](#license)

## What it does

Each search term runs inside each subreddit you list through Arctic Shift, or across all of Reddit with your own Serper or Brave key. By default a result is kept only when the search term is in the post's title or text, so loosely related results are dropped and not charged. Subreddits with no search term return their newest posts. Thread URLs return the post and its top comments. Every row names its source in `data_route`, and each query also returns one free summary row with its counts. No Reddit account, login, or Reddit API key is used, and no RSS feed is read.

## Routes and fallback

| Route | `data_route` | What it reads | Needs | Comments |
|---|---|---|---|---|
| Arctic Shift (default) | `arctic_shift` | A free public archive of Reddit run by an independent open source project, about 15 minutes behind live Reddit | A subreddit for every search | Yes |
| Your search key | `search_serper` or `search_brave` | `site:reddit.com` results from Serper (Google) or Brave Search, each post then read through Arctic Shift for its text | Your own Serper or Brave key | Only when Arctic Shift has the post |
| Reddit embed | `reddit_embed` | A thread Arctic Shift has not stored yet: title and score from Reddit's public embed page, without text | Nothing | No |

`reddit_route` picks the main route: `arctic_shift` (default), `search_key`, or `both`. If the main route fails or cannot run, the actor switches to the other route when it can and says so in `route_note`. A search with no subreddit and no key returns one free row that says what to add. Your key goes only to the provider you picked, in a request header, and is never stored, logged, or written to a row. Search calls are billed to you by your provider.

All of the work runs on Apify. This package is a thin client that starts the actor run, waits for it, and hands back the dataset rows.

## Quick start

You need Node.js 18 or newer and an Apify account with an API token.

Add this to your Claude Desktop config (`claude_desktop_config.json`), Cursor, or any MCP client:

```json
{
  "mcpServers": {
    "mamba-reddit-search-thread-scraper": {
      "command": "npx",
      "args": [
        "-y",
        "@mambalabsdev/mcp-reddit-search-thread-scraper"
      ],
      "env": {
        "APIFY_TOKEN": "your-apify-token"
      }
    }
  }
}
```

Restart the client. The `search_reddit` tool appears.

## Prerequisites

- Node.js 18 or newer.
- An Apify API token from [console.apify.com/account/integrations](https://console.apify.com/account/integrations), set as `APIFY_TOKEN`. The server lists its tool without a token; a call needs one.

## Example prompts

- "Find Reddit posts from the last week in r/sales and r/startups that mention cold email."
- "Search all of Reddit for posts asking for a cold email agency, using my Serper key."
- "Pull the top comments from this Reddit thread into a table."
- "What are people on r/SaaS posting about this month?"

## Tool

`search_reddit`. One tool. Example arguments:

```json
{
  "search_terms": [
    "cold email"
  ],
  "subreddits": [
    "sales"
  ],
  "since_days": 7,
  "max_posts_per_query": 50
}
```

## Inputs

Every buyer facing input of the actor. Omitted inputs take the actor's own defaults.

| Input | Type | Required | Description |
|---|---|---|---|
| `search_terms` | array | no | Keywords to search Reddit for. With subreddits, each term runs inside each subreddit. |
| `subreddits` | array | no | Subreddits to search in, or to read when there is no search term. |
| `thread_urls` | array | no | Thread URLs to read with their comments, up to 200. |
| `reddit_route` | string | no | Main route: arctic_shift (default, needs a subreddit), search_key (your key, all of Reddit), or both. The other route takes over when the main one fails. |
| `search_provider` | string | no | Provider of your key: serper or brave. Default serper. |
| `search_api_key` | string | no | Your own Serper or Brave key, for keyword search across all of Reddit. Never stored or logged. |
| `read_current_state` | boolean | no | Read each post's current title, score, and comment count from Reddit's embed page. Default true. |
| `sort` | string | no | Search order: new, top, or comments (by stored score or comment count); relevance and hot read as new. Default new. |
| `subreddit_sort` | string | no | Subreddit read order: new, or top by stored score; hot and rising read as new. Default new. |
| `time_window` | string | no | For searches, only posts from the last hour, day, week, month, year, or all. Default month. |
| `since_days` | integer | no | Drop posts older than this many days. |
| `max_posts_per_query` | integer | no | Posts per search or subreddit, up to 1,000. Default 50. |
| `require_term_match` | boolean | no | Keep a search result only when the term is in its title or text. Default true. |
| `include_comments` | boolean | no | Also return the comments of every post found. Default false. |
| `max_comments_per_thread` | integer | no | Comments per thread, top first, up to 500. Default 25. |
| `comment_depth` | integer | no | Reply levels under each top level comment, 1 to 10. Default 3. |
| `only_new` | boolean | no | Skip posts and comments an earlier run returned for the same query. Default false. |
| `state_key` | string | no | Name for the only new results memory. |
| `max_charge_usd` | string | no | Optional cost cap for the run. |
| `proxy_mode` | string | no | auto or residential, for the embed page reads. A reliability setting; it does not get around Reddit's block. Default auto. |

## Pricing

Pay per event on Apify, billed to your Apify account. Prices in USD.

| Event | Free plan | Bronze | Silver | Gold |
|---|---|---|---|---|
| `post-row`, per Reddit post | $0.0015 | $0.001425 | $0.00135 | $0.001275 |
| `comment-row`, per Reddit comment | $0.0015 | $0.001425 | $0.00135 | $0.001275 |
| `apify-actor-start`, per run start | $0.003144 | $0.003144 | $0.003144 | $0.003144 |

Summary rows, refused input, a query no route could answer, search results without the term, and already delivered rows are free. A blocked run charges only the start fee. Mamba Labs pays the platform costs of the run, Apify proxy traffic included. Calls on your own search key are billed to you by your provider.

## FAQ

**Why did runs return 0 posts in November 2026?** Reddit closed logged out search on about 5 November 2026 and ends RSS on 13 November 2026. The actor's version 0.1 read those feeds. Version 0.2 reads Arctic Shift and, with your key, a search engine. Name a subreddit, or pass your search key.

**Can I search all of Reddit without naming a subreddit?** Yes, with your own Serper or Brave key in `search_api_key`. Arctic Shift cannot search without a subreddit.

**What does a run cost when Reddit data cannot be reached?** Only the start fee.

**Why is a run slow sometimes?** Arctic Shift is busy: on 12 November 2026 about 2 of 3 keyword searches were refused on the first try and answered on a retry. Each call is retried 5 times before the actor switches route.

## Version notes

**1.1.0, 12 November 2026.** Matches actor version 0.2. Tool inputs: adds `reddit_route`, `search_provider`, `search_api_key`, and `read_current_state`; every earlier input still works. Output: rows add `score`, `comment_count`, `removal_state`, `data_route`, `embed_checked`, and `route_note`. Charges: unchanged.

**1.0.0.** First release.

## Full actor documentation

[apify.com/mambalabs/reddit-search-thread-scraper](https://apify.com/mambalabs/reddit-search-thread-scraper)

## License

MIT. See [LICENSE](LICENSE).
