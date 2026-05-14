# Dynamic Absurd Catalog Website

## Goal

Create a novelty website that is dynamically generated per session. On first load, the user chooses topic chips such as shopping, guitars, musical instruments, technology, news, and celebrity news. The app then searches and gathers related content, builds a strange explorable website from the results, and lets the user reset to generate a new session.

The intended tone is an absurd catalog: a chaotic fake store, newsstand, wiki, and media browser mashed together. It should look intentionally wild while still being usable.

## Target Architecture

Build a local demo as a Vite React frontend with a small Node/Express API.

- Frontend: Vite, React, TypeScript, CSS modules or plain CSS.
- Backend: Node, Express, TypeScript.
- Runtime target: local development first.
- Secrets: stored server-side in `.env`, never exposed to the client bundle.
- Default behavior: live providers when keys are available, graceful mock/fallback content when they are not.

## API Shape

### `GET /api/topics`

Returns starter topics and provider availability.

Example response:

```json
{
  "topics": [
    "shopping",
    "guitars",
    "musical instruments",
    "technology",
    "news",
    "celebrity news"
  ],
  "providers": {
    "wikipedia": "available",
    "youtube": "missing-key",
    "search": "mock"
  }
}
```

### `POST /api/sessions`

Accepts selected topics and returns a generated content universe.

Request:

```json
{
  "topics": ["guitars", "technology", "celebrity news"],
  "seed": "optional-seed"
}
```

Response:

```json
{
  "sessionId": "uuid",
  "seed": "resolved-seed",
  "theme": {
    "name": "Absurd Catalog",
    "palette": ["#fffb00", "#ff3df2", "#00e0ff", "#111111"],
    "motion": "high"
  },
  "modules": [],
  "items": [],
  "providerStatus": {},
  "warnings": []
}
```

## Shared Content Model

Normalize all provider results into a single `ContentItem` shape:

```ts
type ContentItem = {
  id: string;
  kind: "wiki" | "video" | "image" | "shopping" | "news" | "generated";
  title: string;
  snippet?: string;
  imageUrl?: string;
  videoId?: string;
  price?: string;
  url?: string;
  sourceName: string;
  attribution?: string;
  fetchedAt: string;
  generated: boolean;
};
```

## Provider Plan

Use provider adapters so the app is not locked to one search service.

- Wikipedia: use MediaWiki REST search by default. This can work without a paid key.
- YouTube: use YouTube Data API when `YOUTUBE_API_KEY` exists.
- Web, image, and shopping search: support adapters for Apify, SerpAPI or Brave-style providers, and existing Google Custom Search credentials.
- Google result pages should not be scraped directly. Use provider APIs for Google-like web, image, and shopping surfaces.
- Aggressive scraping applies to destination URLs returned by providers, with source attribution, short excerpts, timeouts, caching, and failure handling.
- If shopping data is unavailable, generate fake catalog/ad cards from other content and mark them as generated.

Suggested environment variables:

```env
YOUTUBE_API_KEY=
APIFY_TOKEN=
APIFY_ACTOR_ID=
SERPAPI_KEY=
BRAVE_SEARCH_API_KEY=
GOOGLE_CSE_API_KEY=
GOOGLE_CSE_CX=
```

## Frontend Behavior

- First screen shows topic chips and a generate button.
- Users can choose one or more topics.
- The generated site should use a fresh session seed.
- Reset button discards the current session and calls `POST /api/sessions` again.
- Keep the current session in browser state or `sessionStorage` so refresh does not immediately erase it.
- Show provider warnings when content falls back to mocks or generated cards.
- All external content should link back to its source.

## Visual Direction

Default style: absurd catalog.

Expected modules:

- Warped product shelves.
- Fake price tags for generated items.
- Wiki rabbit-hole cards.
- Rumor/news tickers.
- Video walls.
- Fake ads clearly marked as generated when not sourced from a real provider.
- Strange sort controls and fake catalog departments.
- Bright, clashing colors and animated elements.

The page can be chaotic, but text and controls should remain readable and usable on desktop and mobile.

## Test Plan

- Unit test provider normalization, query generation, provider selection from env vars, fallback behavior, and scraper truncation.
- API test successful session generation, partial provider failure, no-key fallback mode, and reset/new-session behavior.
- React test topic selection, loading states, warnings, generated labels, and reset.
- Playwright smoke test first load, topic selection, generated catalog rendering, reset, desktop layout, and mobile layout.
- Run `npm run build` before handoff.
- Run `npm test` before handoff.

## Source Notes

- Google Custom Search JSON API can return web/image JSON results but is closed to new customers and existing customers transition by January 1, 2027: https://developers.google.com/custom-search/v1/overview
- YouTube search uses `search.list`, which costs 100 quota units per call: https://developers.google.com/youtube/v3/docs/search/list
- Google Merchant API is for managing Merchant Center product data, not general Google Shopping search: https://support.google.com/merchants/answer/16494522
- Wikipedia content can use MediaWiki REST search endpoints: https://www.mediawiki.org/wiki/API:REST_API/Reference
