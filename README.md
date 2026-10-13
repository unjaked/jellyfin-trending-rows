
# Jellyfin JS Custom Rows — Platform Hub + TMDB Trending Rows

Custom JavaScript for Jellyfin that adds configurable home-screen rows:

- - **Trending Movies & Shows** — Fetches trending movies and series from [TMDB.org](https://www.themoviedb.org/) and cross-references them against your Jellyfin library. Only shows titles you actually own, ranked 1–25 (configurable up to 50) with poster art, community ratings, and year of release.
- **Platform Hub** — Clickable streaming platform cards (Netflix, Disney+, Apple TV+, Prime Video). Tap one to expand a scrollable row of all your content tagged with that platform.
- **Franchise Hub** — Clickable franchise cards such as Marvel, Star Wars, DC, Pixar, Harry Potter, and Pirates of the Caribbean. Click a franchise to expand a row of library items tagged with that franchise.

Fully client-side, just a single JavaScript injection.

Designed to work on any theme, but certain themes may require some manual adjustment.

![Jellyfin Custom Rows Demo](screenshots/demo.gif)

---

## Features

- **Configurable** — Enable or disable Platforms, Franchises, Trending Movies, Trending Shows, row order, and more
- **Works on any theme** — Designed to use Jellyfin's native attributes, allowing themes to make adjustments accordingly.
- **Instant ID matching** — Builds a `Map` of TMDB/IMDB IDs from your entire catalog for O(1) lookups. No sequential API calls per trending item.
- **Fallback matching** — If provider IDs are missing, falls back to title + year comparison.
- **Responsive** — Grid layout on tablet, compact cards on mobile.
- **Lightweight** — Single IIFE, no dependencies, no build step.
- **MutationObserver injection** — Waits for the home screen DOM to be ready. No polling or intervals.

---

## Requirements

- Jellyfin **10.8+** (web client) (Works on 12!)
- [Jellyfin JavaScript Injector Plugin](https://github.com/n00bcodr/Jellyfin-JavaScript-Injector)
- A free [TMDB.org API key](https://developer.themoviedb.org/docs/getting-started) (TMDB > Account > Settings > API > Copy your API key)
- Jellyfin tags on your content tagged for platform and franchise rows

---

## Installation

### 1. Create a TMDB API app

Go to [themoviedb.org/settings/api](https://www.themoviedb.org/settings/api), scroll down, and copy the **API Key** (NOT the "API Read Access Token").

### 2. Add the script to Jellyfin

This script integrates with the [Jellyfin-JavaScript-Injector plugin](https://github.com/n00bcodr/Jellyfin-JavaScript-Injector).

1. Install the JS Injector plugin following the instructions in its repository
2. Open your Jellyfin dashboard
3. Go to **Plugins** → **JavaScript Injector**
4. Add a new script and paste the contents of [`jellyfin-custom-rows.js`](jellyfin-custom-rows.js)
5. Replace `YOUR_TMDB_API_KEY` with your actual Client ID
6. Save

### 3. Tag your content

The platform and franchise cards filter your library by Jellyfin tags. Add tags to your movies and series matching the platform they belong to:

| Platform | Tag |
|---|---|
| Apple TV+ | `Apple TV` |
| Disney+ | `Disney Plus` |
| Prime Video | `Amazon Prime Video` |
| Netflix | `Netflix` |

| Franchise | Jellyfin tag |
|---|---|
| Marvel | `marvel` |
| Star Wars | `star wars` |
| DC Universe | `dc universe` |
| Pixar | `pixar` |
| Harry Potter | `harry potter` |
| Pirates of the Caribbean | `pirates of the caribbean` |

*Tip: You can bulk-tag via **Edit Metadata → Tags** or use external tools.*
### 4. Reload

Hard-refresh your browser (`Ctrl+Shift+R` / `Cmd+Shift+R`). The new rows will appear below the Spotlight section on your home screen.

---

## Configuration

Most configuration is at the top of `jellyfin-custom-rows.js`.

```js
const TMDB_API_KEY = "YOUR_TMDB_API_KEY";

const SHOW_RATING = true;
const SHOW_TRENDING_MOVIES = true;
const SHOW_TRENDING_SHOWS = true;
const SHOW_PLATFORMS = true;
const SHOW_FRANCHISES = true;
const SHOW_TRENDING_RANK_NUMBERS = true;
const LIMIT = 50;

const ROW_ORDER_TRENDING_MOVIES = 1;
const ROW_ORDER_TRENDING_SHOWS = 2;
const ROW_ORDER_PLATFORMS = 3;
const ROW_ORDER_FRANCHISES = 4;
```

### Row visibility

Use these flags to enable or disable rows:

| Option | Description |
|---|---|
| `SHOW_PLATFORMS` | Shows or hides the Platforms row |
| `SHOW_FRANCHISES` | Shows or hides the Franchises row |
| `SHOW_TRENDING_MOVIES` | Shows or hides the TMDB Trending Movies row |
| `SHOW_TRENDING_SHOWS` | Shows or hides the TMDB Trending Shows row |

Example:

```js
const SHOW_PLATFORMS = true;
const SHOW_FRANCHISES = true;
const SHOW_TRENDING_MOVIES = true;
const SHOW_TRENDING_SHOWS = false;
```

### Trending display options

| Option | Description |
|---|---|
| `SHOW_RATING` | Shows or hides the community rating next to the year |
| `SHOW_TRENDING_RANK_NUMBERS` | Shows or hides numbered rank badges |
| `LIMIT` | Controls how many TMDB trending items are fetched and matched |

Example:

```js
const SHOW_RATING = false;
const SHOW_TRENDING_RANK_NUMBERS = true;
const LIMIT = 50;
```

### Adding or removing platforms and franchises

Edit the `STUDIOS` or `FRANCHISES` arrays at the top of the script.

Each entry supports:

```js
{
    name: "HBO Max",              // Display name
    tag: "HBO Max",               // Jellyfin tag to filter by
    gradient: "linear-gradient(135deg,#1a0a2e 0%,#0d0018 100%)",
    logo: "https://example.com/hbo-logo.png",
    invert: false,                // Optional: true if the logo should be inverted
    big: false                    // Optional: true for logos that need larger sizing
}
```

The `tag` value must match the tag on your Jellyfin items.

### Changing row order

Rows are added in the `injectUI()` function. To change their order, reorder these blocks:

```js
if (SHOW_PLATFORMS) {
    wrapper.appendChild(buildPlatformSection());
}

if (SHOW_FRANCHISES) {
    wrapper.appendChild(buildFranchiseSection());
}

if (SHOW_TRENDING_MOVIES) {
    wrapper.appendChild(buildTop10Section("Trending Movies", "movie"));
}

if (SHOW_TRENDING_SHOWS) {
    wrapper.appendChild(buildTop10Section("Trending Shows", "tv"));
}
```

## How it works

```
TMDB.org API ──► Trending list (25 items)
                        │
                        ▼
Jellyfin API ──► Full catalog ──► Map<TMDB_ID|IMDB_ID, Item>
                                         │
                                         ▼
                                  Cross-reference
                                         │
                                         ▼
                              Top 25 matched items
                              rendered on home screen
```

1. Fetches the trending list from TMDB's public API (no OAuth needed, just the Client ID).
2. Loads your full Jellyfin catalog for movies or series in a single API call.
3. Indexes all items by their TMDB and IMDB provider IDs into a `Map`.
4. Iterates through trending items and looks up each one in the map — O(1) per item.
5. If no ID match is found, falls back to exact title + year (±1 year tolerance).
6. Renders the first 10 matches with Jellyfin poster art and metadata.

---

## FAQ

**Can I hide any of the rows or features?**
Yes, most of the JavaScript is configurable at the top of the script.

**The Top 25 says "No matches found"**
This means none of the current trending titles exist in your library. The script only shows what you have — it doesn't add content.

**Platform or franchise cards say "No content found."**  
Your content is probably missing the matching Jellyfin tag. Open the item in Jellyfin, edit metadata, and add the tag configured in the `STUDIOS` or `FRANCHISES` array.

**Does this work on Jellyfin apps (Android/iOS/TV)?**
No. Custom JavaScript only runs in the web client. However, it will work if your app is a Progressive Web App.

**Is my TMDB API key a secret?**
Yes, do not publish this key anywhere. It should only be accessible to you.

**Can I use this without IMDB?**
Yes — just remove the two `buildTop10Section` lines near the bottom of the script. The Platform Hub will still work independently.

**What happened to the Trakt integration?**
This repo originally used Trakt, but Trakt has since revoked all API keys for free users and now require their premium tier for API access. This repo now uses TMDB, which is truly free and basically works exactly the same.


---

## License

MIT
</details>