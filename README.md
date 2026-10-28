# Reddit Search and Thread Scraper MCP Server

[![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-reddit-search-thread-scraper)](https://www.npmjs.com/package/@mambalabsdev/mcp-reddit-search-thread-scraper) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-reddit-search-thread-scraper)](https://github.com/mambalabsdev/mcp-reddit-search-thread-scraper/blob/main/LICENSE)

An MCP server for the Mamba Labs Reddit Search and Thread Scraper actor on Apify. Search Reddit by keyword, read subreddits, or read threads with their comments, and get one flat row per post and comment.

## What's Inside

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Prerequisites](#prerequisites)
- [Example prompts](#example-prompts)
- [Tool](#tool)
- [Inputs](#inputs)
- [Pricing](#pricing)
- [Full actor documentation](#full-actor-documentation)
- [License](#license)

## What it does

Each search term runs across all of Reddit or inside each subreddit you list, sorted by new, relevance, top, most comments, or hot. By default a result is kept only when the search term is in the post's title or text, so loosely related results are dropped and not charged. Subreddits with no search term return their newest, hot, top, or rising posts. Thread URLs return the post and its top comments. Each query also returns one free summary row with its counts. Reddit is read through its public feeds; no Reddit account, login, or API key is used.

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
| `sort` | string | no | Search order: new, relevance, top, comments, hot. Default new. |
| `subreddit_sort` | string | no | Subreddit read order: new, hot, top, rising. Default new. |
| `time_window` | string | no | Reddit's time filter: hour, day, week, month, year, all. Default month. |
| `since_days` | integer | no | Drop posts older than this many days. |
| `max_posts_per_query` | integer | no | Posts per search or subreddit, up to 1,000. Default 50. |
| `require_term_match` | boolean | no | Keep a search result only when the term is in its title or text. Default true. |
| `include_comments` | boolean | no | Also return the comments of every post found. Default false. |
| `max_comments_per_thread` | integer | no | Comments per thread, top first, up to 500. Default 25. |
| `comment_depth` | integer | no | Reply levels under each top level comment, 1 to 10. Default 3. |
| `only_new` | boolean | no | Skip posts and comments an earlier run returned for the same query. Default false. |
| `state_key` | string | no | Name for the only new results memory. |
| `max_charge_usd` | string | no | Optional cost cap for the run. |
| `proxy_mode` | string | no | auto or residential. Default auto. |

## Pricing

Pay per event on Apify, billed to your Apify account. Prices in USD.

| Event | Free plan | Bronze | Silver | Gold |
|---|---|---|---|---|
| `post-row`, per Reddit post | $0.0015 | $0.001425 | $0.00135 | $0.001275 |
| `comment-row`, per Reddit comment | $0.0015 | $0.001425 | $0.00135 | $0.001275 |
| `apify-actor-start`, per run start | $0.0005 | $0.0005 | $0.0005 | $0.0005 |

Summary rows, refused input, search results without the term, and already delivered rows are free. Mamba Labs pays the platform costs of the run, Apify proxy traffic included.

## Full actor documentation

[apify.com/mambalabs/reddit-search-thread-scraper](https://apify.com/mambalabs/reddit-search-thread-scraper)

## License

MIT. See [LICENSE](LICENSE).
