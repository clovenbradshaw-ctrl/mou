# Feed Management System

This document explains how to manage social media feeds in the Nashville Community Safety Network website.

## Overview

The feed management system allows you to centrally manage all social media feeds (currently Instagram posts) through a single JSON configuration file. When you add a new feed to this file, it will automatically sync to all pages on the client side when users refresh the page.

## Architecture

### Components

1. **`data/feeds.json`** - Central configuration file that stores all feed data (the "API")
2. **`js/feed-sync.js`** - JavaScript module that fetches and renders feeds dynamically
3. **HTML pages** - Pages that display feeds (index.html, news.html)

### Data Flow

```
User refreshes page
    ↓
FeedSync module fetches feeds.json (with cache: 'no-cache')
    ↓
Creates Instagram embed elements dynamically
    ↓
Instagram's embed.js processes and renders posts
```

## Adding a New Feed

### Step 1: Get the Instagram Post ID

From an Instagram post URL like:
```
https://www.instagram.com/p/DRft-3JgUzX/
```

The post ID is: `DRft-3JgUzX`

### Step 2: Add to feeds.json

Open `data/feeds.json` and add a new entry to the appropriate section:

```json
{
  "version": "1.0",
  "lastUpdated": "2025-12-15T00:00:00Z",
  "feeds": {
    "homepage": [
      {
        "id": "YOUR_POST_ID",
        "platform": "instagram",
        "url": "https://www.instagram.com/p/YOUR_POST_ID/",
        "source": "Organization Name",
        "addedDate": "2025-12-15"
      }
    ]
  }
}
```

**Fields:**
- `id` (required): Instagram post ID
- `platform` (required): Always "instagram" for now
- `url` (required): Full Instagram post URL
- `source` (optional): Organization/account name for documentation
- `addedDate` (optional): Date when feed was added (YYYY-MM-DD)

### Step 3: Update version and lastUpdated

```json
{
  "version": "1.1",
  "lastUpdated": "2025-12-15T12:30:00Z",
  ...
}
```

### Step 4: Test

1. Save the `feeds.json` file
2. Refresh the page where the feed should appear
3. The FeedSync module will fetch the updated configuration (cache: 'no-cache' ensures fresh data)
4. New feed will appear automatically

## Feed Sections

The `feeds.json` file has different sections for different pages:

- **`homepage`** - Feeds displayed on index.html in the "What Social Media is Saying" section
- **`news`** - Feeds displayed on news.html in the coverage section

You can add feeds to the appropriate section based on where you want them to appear.

## How It Works

### Client-Side Sync on Refresh

When a user refreshes any page:

1. The `FeedSync` class initializes on `DOMContentLoaded`
2. It calls `fetchFeeds()` with `cache: 'no-cache'` to get fresh data
3. The module reads the appropriate feed section (e.g., 'homepage' or 'news')
4. For each feed, it creates an Instagram blockquote element
5. Instagram's embed.js script processes and renders the posts
6. Feed statistics are logged to the browser console

### No Backend Required

This system works entirely client-side:
- The JSON file acts as a simple "API"
- No server-side processing needed
- Feeds update on every page refresh
- Cache-busting ensures users always get the latest feeds

## Debugging

### Check Feed Stats

Open the browser console and look for:
```
[FeedSync] Successfully fetched feeds (version 1.0)
[FeedSync] Rendered 6 feeds in homepage-feeds
[Feed Stats] { version: "1.0", lastUpdated: "...", ... }
```

### Common Issues

**Feeds not showing up:**
- Check browser console for errors
- Verify JSON syntax in `feeds.json`
- Ensure Instagram embed script is loaded: `<script async src="//www.instagram.com/embed.js"></script>`
- Check that container IDs match (`homepage-feeds`, `news-feeds`)

**Old feeds still showing:**
- Hard refresh the page (Ctrl+Shift+R or Cmd+Shift+R)
- Check that `cache: 'no-cache'` is set in fetch request
- Verify `lastUpdated` timestamp was updated in feeds.json

## Example: Adding a New Homepage Feed

```json
{
  "version": "1.2",
  "lastUpdated": "2025-12-15T14:00:00Z",
  "feeds": {
    "homepage": [
      // ... existing feeds ...
      {
        "id": "NEW_POST_ID",
        "platform": "instagram",
        "url": "https://www.instagram.com/p/NEW_POST_ID/",
        "source": "Nashville Activists",
        "addedDate": "2025-12-15"
      }
    ],
    "news": [
      // ... news feeds ...
    ]
  }
}
```

Save the file, refresh index.html, and the new feed will appear!

## Future Enhancements

Possible improvements to the system:
- Add support for other platforms (Twitter, Facebook)
- Implement periodic auto-refresh (e.g., every 60 minutes)
- Add feed ordering/priority
- Include manual refresh button
- Add feed categories/tags
- Implement feed analytics tracking
