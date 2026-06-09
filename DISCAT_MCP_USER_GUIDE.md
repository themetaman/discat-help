# DisCat MCP User Guide

---

## Overview

DisCat MCP connects your DisCat vinyl collection manager to Claude Desktop, letting you have natural language conversations about your Discogs collection and wantlist. Instead of clicking through screens, you just ask — and Claude handles the rest.

**What that looks like in practice:**

*"Which sellers have the most of my wantlist items?"*
*"Compare Seller 1 and Seller 2 for Techno, VG+ condition only."*
*"Show me all my wantlist wants that are scanned but nothing's for sale right now."*
*"Give me a breakdown of my collection by genre and style."*

Claude uses a set of DisCat tools behind the scenes — you don't need to know which tool does what, just ask your question in plain English.

### What You Can Do

- **Browse your collection** — search by artist, label, year, genre, style, format, or folder
- **Analyse your wantlist** — see scan states, style breakdowns, and which wants have active listings
- **Plan a buying run** — find which sellers cover the most of your wants, compare two sellers side-by-side, filter by style or condition
- **Spot gaps** — identify wants with no listings, find duplicate releases, see what's uncovered after your top sellers
- **Export results** — save any query output to CSV, JSON, or Markdown

### Prerequisites

Before using DisCat MCP, you need:

1. **DisCat installed and configured** — with your Discogs OAuth credentials set up
2. **Your collection downloaded** — run **⬇️ Download Collection** in DisCat at least once
3. **Claude Desktop installed** — with `DisCatMCP.exe` configured as an MCP server
4. **Your wantlist scanned** — for seller coverage and buying-run features, run **Scan Market** in DisCat first to populate marketplace data

**Note:** The MCP tools work from DisCat's local database — they don't call the Discogs API in real time (except **Sync My Wantlist**). If your data feels stale, re-run the relevant DisCat operation first, then come back to Claude.

---

## Getting Started

### What You Need Before Your First Query

Before Claude can answer questions about your collection, three things need to be in place:

**1. DisCat is running and your data is fresh**

DisCat doesn't need to be open for MCP to work — but the data in its database needs to be up to date. If you've recently added records to your collection or wantlist on Discogs, run the relevant sync in DisCat first:

- New collection items → **⬇️ Download Collection**
- New wantlist items → **Sync My Wantlist**
- Fresh marketplace listings → **Scan Market**

**2. Claude Desktop is running**

Open Claude Desktop normally. As long as `DisCatMCP.exe` is configured in your Claude Desktop settings, it starts automatically in the background — you don't need to do anything else.

**3. The MCP connection is active**

You can confirm it's working by checking the DisCat **Settings** tab — the MCP status indicator shows whether Claude Desktop has an active connection. Alternatively, just ask Claude:

*"Are you connected to DisCat?"*

Claude will either respond with your collection stats or let you know it can't reach the tools.

---

### Your First Query

Once everything is connected, start simple:

*"Give me an overview of my collection."*

Claude will call `collection_stats` and come back with your total item count, folder breakdown, format split, purchase totals, and genre/style breakdown — a solid confirmation that everything is working.

From there, try your wantlist:

*"Give me a summary of my wantlist."*

This confirms your wantlist data is populated and shows you how many items have been scanned for marketplace listings.

**Tip:** You don't need to phrase questions in any particular way — Claude understands natural language. "How many records do I own?" works just as well as "Show me collection stats."

---

### If Something Doesn't Work

If Claude says it can't access your DisCat data:

1. Check the MCP status in DisCat **Settings** — look for a green indicator
2. Restart Claude Desktop — it sometimes drops the MCP connection after idle periods
3. Make sure `DisCatMCP.exe` hasn't been blocked by antivirus software
4. Check that your `config.env` file has valid Discogs credentials

**Note:** Claude Desktop automatically closes idle MCP connections after a couple of minutes — this is normal. It reconnects on your next query without any action needed.

---

## Core Concepts

You don't need to understand how DisCat works under the hood to use it with Claude — but a few key ideas will help you get better answers and avoid confusion.

### How the Tools Fit Together

Behind the scenes, Claude has access to a set of DisCat tools. Each tool does one specific thing — search the collection, list wantlist items, calculate seller coverage, and so on. When you ask a question, Claude picks the right tool (or combination of tools) automatically.

The tools read from **DisCat's local database** by default — the same data you see in the DisCat app. A few tools — **Sync My Wantlist** and **Scan Market** — do reach out to Discogs live, but only when you specifically ask for them.

Think of it like this:

```
Your question → Claude picks the right tool(s) → DisCat database → Claude explains the results
```

For more complex questions, Claude may chain several tools together — for example, calling `wantlist_styles` to find relevant style tags, then passing those into `seller_coverage` to find the best sellers for that style.

---

### Understanding Scan States

When you run **Scan Market** in DisCat, it scrapes Discogs marketplace listings for each item on your wantlist. Every wantlist item ends up in one of three states:

| State | What it means |
|---|---|
| **Pending** | Never been scanned — we don't know if copies exist |
| **For Sale** | Scanned and copies are currently listed on Discogs |
| **Not Listed** | Scanned but no copies are currently for sale |

**Note:** "Not Listed" is not the same as "impossible to find" — it just means nothing was available at the time of the last scan. Listings come and go. Items in this state are worth setting up Discogs alerts for.

For the buying-run tools (`seller_coverage`, `seller_compare`) to work well, you want as few **Pending** items as possible. Run **Scan Market** in DisCat to clear them.

---

### A Note on Discogs Style Tags

DisCat's style filtering is only as good as Discogs' own metadata — and Discogs tagging is inconsistent.

The most common example: a record you'd call **Deep House** may simply be tagged **House** on Discogs, with no Deep House sub-tag at all. If you filter by `style="Deep House"` and get surprisingly few results, that's almost certainly why.

The fix is to search broader:

*"Which sellers cover the most of my House wants?"*

Then use a comma-separated list to cast a wider net:

*"Compare Seller 1 and Seller 2 for House, Deep House, Chicago House, and Garage House combined."*

**Tip:** Run `wantlist_styles` first — it lists every style tag actually present in your wantlist with counts. This tells you exactly which style names are worth filtering by before you run a coverage query.

---

### Your Data Is a Snapshot

DisCat's database reflects the last time you synced. Marketplace listings are cached with a time limit — so seller coverage results may not reflect what's listed on Discogs right now.

If you're about to make buying decisions, it's worth refreshing first:

- **Scan Market** → updates marketplace listings for your wantlist
- **Sync My Wantlist** → pulls any new additions from Discogs
- **⬇️ Download Collection** → updates your collection data

---

## Your Collection

These tools let you explore and interrogate your Discogs collection from within Claude.

### Getting an overview — `collection_stats`

*"Give me an overview of my collection."*
*"How much have I spent in total?"*
*"What's my genre breakdown?"*

`collection_stats` returns a summary of everything in your database:

- Total item count and folder breakdown
- Format split (12", LP, CD, etc.)
- Purchase totals — total spend, number of purchases matched to collection items
- Genre and style breakdowns, sorted by count

**Note:** Genre and style data is only populated for items downloaded with the **Genres & Styles** option ticked in DisCat. The `items_with_genre_data` field tells you how much of your collection has this data.

---

### Searching your collection — `collection_search`

*"Show me all my Jazz records from 1960 to 1975."*
*"Which House records do I own on D.J. International?"*
*"What's my most expensive record?"*
*"Find everything in my Favourites folder."*

`collection_search` is the main collection browsing tool. All filters are AND-combined — you can mix and match freely:

| Filter | What it does | Example |
|---|---|---|
| `query` | Artist or title keyword | `"Roy Ayers"` |
| `folder` | Exact folder name | `"Favourites"` — use `folder_list` first |
| `label` | Record label (partial match) | `"Blue Note"` |
| `year` | Exact release year | `1987` |
| `year_from` / `year_to` | Year range | `1980` to `1989` |
| `format` | Format type (partial match) | `"12\""`, `"LP"` |
| `catno` | Catalog number (partial match) | `"BN-4003"` |
| `country` | Country of release | `"Germany"` |
| `genre` | Genre tag | `"Electronic"` |
| `style` | Style/sub-genre tag | `"Deep House"` |
| `sort_by` | Sort order | `price_desc`, `artist`, `year` |

**Note:** `country`, `genre`, and `style` filtering requires detailed metadata. If you haven't downloaded with those options, these filters will return no results.

**Finding your most expensive records:**

*"What are my most expensive GBP purchases?"*

Because purchases are stored in mixed currencies, always tell Claude which currency you want when asking about prices — this lets it filter correctly before sorting.

---

### Drilling into a single record — `collection_item`

*"Give me full details on that Roy Ayers record."*
*"What condition did I buy it in? Who did I buy it from?"*

Once you have a `release_id` from `collection_search`, you can pull the complete record: genres, styles, country, tracklist, credits, community have/want counts, and all purchase metadata (price, date, seller, conditions, order link).

If you own the same release twice, use the `instance_id` to target a specific copy.

---

### Price + market source provenance

`collection_search` and `collection_item` results now include a `price_paid` object with `is_estimated=true` when the value came from Auto-Estimate or bulk CSV import (raw string ends with `(e)`) rather than a real Discogs purchase order. `market_values` (both `stats` and `valuation` modes), `collection_item.market_stats`, and `collection_valuation` per-item rows include a `source` enum: `'sales'` (canonical Discogs sales history), `'listings'` (asking-price fallback when no sales history exists — less reliable), or `'manual'` (user override). `valuation_rules.data_provenance` breaks down item counts and dollar value share by source so the LLM can flag fragile estimates. Use these signals to ask things like *"what's my collection worth and how confident should I be?"* — Claude will weight sales-history rows more heavily and call out outlier manual or listings-derived values.

---

### Ishkur Guide — `ishkur_genre` and `ishkur_for_release`

Bundled genre reference data from *Ishkur's Guide to Electronic Music* — 156 genres with long-form descriptions, scene/era, aliases, and exemplar tracks (title/url/year). `ishkur_genre(name)` looks up a genre directly (case-insensitive, with comma-aware alias fallback via the `also` field). `ishkur_for_release(release_id)` walks a collection release's Discogs styles and returns one Ishkur match per style, deduplicated. Use for questions like *"what is acid house"*, *"tell me about the genre lineage on this record"*, or *"give me three records in my collection that match Ishkur's definition of detroit techno"*. Data lives in `discat.db` and refreshes only on EXE rebuild — `ISHKUR_DATA_VERSION` controls the rollout independently of the schema.

---

### Finding duplicates — `find_duplicates`

*"Do I have any duplicate releases?"*
*"Which records do I own more than one copy of?"*

Scans your entire collection in a single query and returns any release_id that appears more than once — along with which folders the copies are in and their instance IDs. Useful before doing a clear-out or folder reorganisation.

---

### Browsing by folder — `folder_list`

*"What folders do I have?"*
*"How many records are in each folder?"*

Returns all your Discogs folders with item counts, sorted largest first. Always run this before passing a folder name to `collection_search` — folder names are case-sensitive exact matches.

---

### Custom fields — `list_custom_fields` and `custom_field_search`

*"Which records are marked for upgrade?"*
*"Show me everything with a custom field value."*

If you use Discogs custom fields (e.g. **Upgrade**, **Price Paid**, **Sleeve Condition**), DisCat stores them locally and makes them searchable.

1. **`list_custom_fields`** — lists every custom field name currently in your database
2. **`custom_field_search`** — finds all records with a given field value

Example: *"Show me all records tagged Upgrade=yes"* → Claude calls `custom_field_search(field_name="Upgrade", field_value="yes")`.

---

### Purchase history — `purchase_history`

*"What did I buy from Seller 1?"*
*"Show me my most expensive EUR purchases."*
*"What did I buy last month?"*

Returns your purchase orders with full metadata: date, artist, title, price, currency, seller, media condition, sleeve condition, and order status.

**Important — currencies are mixed:** Discogs records purchases in the seller's local currency. If you ask for most expensive purchases without specifying a currency, you'll get raw numbers compared across GBP, EUR, USD, etc. — which is meaningless. Always filter by currency first:

*"Show me my most expensive GBP purchases."*
*"What did I spend in EUR last year?"*

---

## Your Wantlist

These tools let you explore your wantlist and understand what you're looking for and how much of it is currently available on the market.

### Wantlist summary — `wantlist_stats`

*"Give me a summary of my wantlist."*
*"How many of my wants have been scanned?"*

Returns the full picture at a glance:

| Field | What it means |
|---|---|
| `wantlist_items` | Total items on your wantlist |
| `scanned_releases` | Items where we've checked for marketplace listings |
| `unscanned_releases` | Items never scanned — we don't know if copies exist |
| `releases_with_listings` | Scanned and copies are currently for sale |
| `releases_with_no_listings` | Scanned but nothing for sale right now |
| `top_artists` | Your top 10 most-wanted artists |
| `year_min` / `year_max` | Year range of your wants |
| `total_cached_listings` | Total marketplace listings across all scanned wants |

---

### Style and genre tags — `wantlist_styles`

*"What styles make up my wantlist?"*
*"How much of my wantlist is tagged House vs Deep House?"*

Lists every Discogs style and genre tag across your wantlist with counts, sorted highest first. **Run this before any style-filtered query** — it tells you exactly which style names are worth filtering by and highlights tagging gaps (e.g. if `House` has 80 items but `Deep House` only has 7, most of your House wants aren't sub-tagged).

---

### Searching your wantlist — `wantlist_search`

*"Show me my unscanned Techno wants."*
*"Which of my House wants have copies for sale?"*
*"What 1990s Jungle records am I looking for?"*

Filters your wantlist by any combination of:

- **Text search** — artist or title
- **Style** — comma-separated OR: `"House,Deep House,Chicago House"`
- **Genre** — comma-separated OR: `"Electronic,Jazz"`
- **Year range** — `year_from` / `year_to`
- **Scan state** — `pending`, `for_sale`, or `not_listed`

Each result includes the item's artist, title, year, styles, genres, your rating, and scan state.

**Useful combinations:**

*"Show me all my Techno wants that are For Sale."* → filter `style="Techno"`, `scan_state="for_sale"`

*"Which of my House wants have never been scanned?"* → filter `style="House"`, `scan_state="pending"`

---

### Listing wantlist items — `wantlist_list`

*"List all my wantlist items."*
*"Which of my wants have cached listings?"*

Returns every item on your wantlist. The `has_cached_listings` field tells you at a glance which items have been scanned and have marketplace data ready — without having to look up each one individually.

---

### Syncing your wantlist — `wantlist_sync`

*"Sync my wantlist."*
*"Pull the latest changes from Discogs."*

Equivalent to pressing **Sync My Wantlist** in DisCat. Calls the Discogs API to refresh your local wantlist — picks up items you've added or removed since the last sync.

Run this if you've updated your wantlist on Discogs recently and want the changes reflected before running coverage queries.

---

## Planning a Buying Run

This is where DisCat MCP really earns its keep. If you're heading to a record fair, planning an online shopping session, or trying to work out which sellers are worth visiting, these tools help you get the most from your time and budget.

### Finding the best sellers — `seller_coverage`

*"Which sellers have the most of my wants?"*
*"Who covers the most of my House wantlist?"*
*"Which sellers have the best coverage of my Techno wants, VG+ condition minimum?"*

Ranks sellers by how many of your wantlist items they stock, and calculates a cheapest-possible total if you were to buy everything they have from them.

**Key parameters:**

| Parameter | What it does |
|---|---|
| `min_media` | Minimum media condition — e.g. `"VG+"` filters out anything below that |
| `min_sleeve` | Minimum sleeve condition |
| `top_n` | How many sellers to return (default 20) |
| `style` | Filter to wants tagged with this style. Comma-separated for OR: `"House,Deep House"` |
| `genre` | Filter to wants tagged with this genre |

**Before you run this:** Make sure your wantlist has been scanned with **Scan Market** in DisCat. Items that haven't been scanned don't appear in seller coverage results at all.

**Tip:** Run `wantlist_styles` first to check which style names are actually in your wantlist before filtering.

---

### Comparing two sellers — `seller_compare`

*"Compare Seller 1 and Seller 2."*
*"Between Seller 1 and Seller 2, who should I prioritise for House?"*
*"What does Seller 1 have that Seller 2 doesn't?"*

Takes two seller names (exactly as they appear in `seller_coverage` output) and breaks down the comparison across four categories:

| Section | What it shows |
|---|---|
| **Overlap** | Items both sellers have — with which is cheaper when same currency |
| **Only in Seller A** | Items unique to the first seller |
| **Only in Seller B** | Items unique to the second seller |
| **Uncovered** | Wants neither seller has |

Each item includes artist, title, styles, price, and currency. The `cheaper` field on overlapping items tells you which seller to buy from when they're selling in the same currency.

Seller names must be exact matches — copy them from `seller_coverage` output.

---

### A full buying-run workflow

Here's a typical buying-run conversation from start to finish:

**Step 1: Check the overall picture**
*"Give me a wantlist summary."*

Check how many items are scanned. If `unscanned_releases` is high, go scan in DisCat first.

**Step 2: Understand your wantlist by style**
*"What styles make up my wantlist?"*

`wantlist_styles` tells you the landscape — which genres and sub-genres to focus on.

**Step 3: Find the top sellers for your target style**
*"Which sellers cover the most of my House wants, VG+ media minimum?"*

`seller_coverage(style="House,Deep House,Chicago House", min_media="VG+")` returns a ranked list.

**Step 4: Decide on your top two**
*"Compare Seller 1 and Seller 2 for House, VG+ minimum."*

`seller_compare` shows you exactly what each has, what overlaps, and what's left uncovered.

**Step 5: Decide your order of attack**
From the overlap section, buy whichever seller is cheaper on duplicate items. Buy unique items from the seller who has more of what you want overall. Add the uncovered items to a watchlist or check other sellers.

**Step 6: Export if you want a reference to work from**
*"Export the seller coverage results to CSV."*

Saves a file you can open in Excel while you're shopping.

---

## Marketplace & Scanning

### Scanning for marketplace listings

*"Scan my wantlist for listings."*

**This operation must be run from DisCat directly** — it cannot be triggered through Claude. Open DisCat and use the **Scan Market** or **Refresh All** button on the Wantlist → Coverage tab. DisCat shows live progress and lets you stop mid-scan.

If you ask Claude to scan, it will give you step-by-step instructions to do it in DisCat, then you can return to Claude for analysis once the scan is complete.

---

### Looking up listings for one release — `marketplace_listings`

*"What copies of release 12345 are listed right now?"*
*"Show me VG+ listings for that release."*

Returns cached marketplace listings for a single wantlist release. You'll need the `release_id` — Claude can get it from `wantlist_list` or `wantlist_search`.

Supports condition filtering (`min_media`, `min_sleeve`) to narrow down results before reviewing.

If the release hasn't been scanned yet, you'll get a `no_cache` status — run **Scan Market** in DisCat first.

---

## Exporting Results

### Saving results to a file — `report_export`

*"Export the seller coverage to CSV."*
*"Save the results as a Markdown file."*
*"Export to JSON and save it to my desktop."*

Exports the seller coverage report to a file you can open outside Claude. Three formats available:

| Format | Best for |
|---|---|
| `csv` | Excel, sorting, filtering — good for shopping lists |
| `json` | Programmatic use, importing into other tools |
| `markdown` | Readable text file, sharing with others |

By default, the file is saved to your DisCat output directory with a standard filename (`wantlist_coverage_report.csv` etc.). You can specify a full path if you want it somewhere else:

*"Export to CSV and save it to C:\Users\theme\Desktop\shopping-list.csv."*

**Note:** `report_export` specifically exports seller coverage data. For ad-hoc collection or wantlist search results, just ask Claude to format its response as a table — or copy-paste directly from the Claude chat.

---

## Discogs Catalogue Tools

Four tools query the full Discogs catalogue (10M+ releases, 2M+ artists, labels, masters) from a local SQLite database — no API calls, instant results. Requires the `discogs-db/discogs_catalogue.db` file to be present alongside the EXE.

| Tool | What it does |
|---|---|
| `catalogue_release` | Full details for any Discogs release ID — tracklist, formats, labels, identifiers |
| `catalogue_pressings` | Find all pressings/versions of a release via its master ID |
| `catalogue_search_catno` | Search the entire Discogs catalogue by partial catalogue number |
| `catalogue_label` | Label lookup by ID or name; optional `include_releases=true` |

**Note:** These tools search the *full Discogs catalogue*, not just your collection. Use them to research releases you don't own yet, identify pressings, or look up label information.

### Market & Valuation Tools

MCP tools for querying market values and estimating collection value.

- **`market_stats`** — look up scraped market data (median/avg/high/low/last sold) for a release in your collection
- **`collection_valuation`** — estimate total collection value using stored market data and condition multipliers

---

## Troubleshooting

### "Claude says it can't access DisCat"

1. Check the MCP status indicator in DisCat **Settings** — it should show green
2. Restart Claude Desktop — idle connections drop after a couple of minutes and reconnect on the next query
3. Check that `DisCatMCP.exe` hasn't been blocked by Windows Defender or antivirus
4. Verify your `config.env` has valid Discogs OAuth credentials (open DisCat and check the Settings tab)

---

### "Seller coverage returns no results"

The most common cause: your wantlist hasn't been scanned yet. Run **Scan Market** in DisCat, then try again.

If you're filtering by style or condition and getting nothing back:

- Check which styles are actually in your wantlist with `wantlist_styles` — the style name may differ from what you expected
- Try broadening your condition filter — `min_media="VG"` instead of `"NM"`
- Try removing the style filter entirely to confirm data is present, then add it back

---

### "Genre/style/country filters on collection search return nothing"

These filters require **detailed metadata** to be downloaded. In DisCat, re-run **⬇️ Download Collection** with the **Genres & Styles** option ticked. The `items_with_genre_data` field in `collection_stats` tells you what percentage of your collection has this data.

---

### "My wantlist stats look stale"

Run **Sync My Wantlist** in DisCat (or ask Claude: *"Sync my wantlist"*) to pull the latest from Discogs. Then re-run your query.

---

### "I edited a record on Discogs but DisCat still shows the old data"

If you update a record on the Discogs website (e.g. change a condition grade or add a note), you need to re-download your collection to pull the change into DisCat — and you must do it **without** the **Use cache** option.

**Why:** "Use cache" reads the JSON that was saved the last time you downloaded. It skips the Discogs API entirely, so any web edits since then won't be picked up.

To get the updated data: run **⬇️ Download Collection** in DisCat with **Use cache** unchecked. This fetches fresh data from Discogs and overwrites the cache.

**Tip:** If you want to make collection edits and keep everything in sync without re-downloading, use DisCat's own **Edit Record** dialog (right-click a record in the Collection tab) — changes are written to the local database and synced to Discogs immediately via the API.

---

### "Prices look wrong or mixed up"

Purchase prices are stored in the seller's local currency (GBP, EUR, USD, etc.) — DisCat doesn't convert them. Always tell Claude which currency you're interested in when asking about prices, e.g.:

*"What are my most expensive GBP purchases?"*

For `seller_coverage` and `seller_compare`, the cheapest total price is calculated per-currency — items in different currencies aren't added together.

---

### "Scan Market opened Chrome but nothing happened"

Discogs requires you to be logged in for marketplace scraping. When Chrome opens, log into Discogs manually — DisCat polls for login for up to 2 minutes before starting the scan. If you miss the window, start the scan again.

---

### "seller_compare says it can't find a seller"

Seller names in `seller_compare` must exactly match how they appear in `seller_coverage` output — including capitalisation and any special characters. Copy the name directly from the coverage results.

---

## Quick Reference

### All Tools at a Glance

| Tool | What it does | Needs scan data? |
|---|---|---|
| `collection_stats` | Collection overview — counts, spend, genre breakdown | No |
| `collection_search` | Search collection by any combination of filters | No (genre/style/country need metadata) |
| `collection_item` | Full details for a single release | No |
| `find_duplicates` | Releases you own more than one copy of | No |
| `folder_list` | All folders with item counts | No |
| `purchase_history` | Your order history with prices and conditions | No |
| `list_custom_fields` | Custom field names in your database | No |
| `custom_field_search` | Find records by custom field value | No |
| `wantlist_stats` | Wantlist summary with scan state breakdown | Scan stats only |
| `wantlist_styles` | All style/genre tags in your wantlist with counts | No |
| `wantlist_search` | Filter wantlist by query/style/genre/year/scan state | No |
| `wantlist_list` | Full wantlist with `has_cached_listings` per item | No |
| `wantlist_sync` | Pull latest wantlist from Discogs API | — (calls API) |
| `marketplace_listings` | Cached listings for one release | Yes (per release) |
| `scan_wantlist` | Scrape fresh listings from Discogs (opens Chrome) | — (scrapes live) |
| `seller_coverage` | Rank sellers by wantlist coverage and total price | Yes |
| `seller_compare` | Compare two sellers side-by-side | Yes |
| `report_export` | Export seller coverage to CSV / JSON / Markdown | Yes |
| `market_stats` | Market value data for a release (median/avg/high/low) | No |
| `collection_valuation` | Estimate total collection value using market data | No |
| `catalogue_release` | Full details for any Discogs release (whole catalogue) | No (needs discogs_catalogue.db) |
| `catalogue_pressings` | All pressings/versions of a release via master ID | No (needs discogs_catalogue.db) |
| `catalogue_search_catno` | Search full catalogue by partial catalogue number | No (needs DuckDB) |
| `catalogue_label` | Label lookup by ID or name | No (needs DuckDB) |

---

### Condition Grades (Discogs Standard)

From best to worst:

| Grade | Abbreviation | What it means |
|---|---|---|
| Mint | M | Unplayed, perfect |
| Near Mint | NM or M- | Essentially perfect |
| Very Good Plus | VG+ | Shows light signs of play, plays perfectly |
| Very Good | VG | Some surface noise, visible marks |
| Good Plus | G+ | Heavy wear, plays through with noise |
| Good | G | Plays but heavily damaged |
| Fair / Poor | F / P | Barely playable |

For buying-run queries, `VG+` is the most common minimum for serious collectors. Condition filters in `seller_coverage` and `marketplace_listings` use these exact abbreviations.

---

### Scan States

| State | Meaning | What to do |
|---|---|---|
| **Pending** | Never been scanned | Run **Scan Market** in DisCat |
| **For Sale** | Scanned — copies currently listed on Discogs | Ready for coverage queries |
| **Not Listed** | Scanned — no copies for sale right now | Set a Discogs alert; check again later |

---

### Example Queries Cheat Sheet

**Collection**
- *"Give me an overview of my collection."*
- *"Show me my Jazz records from the 1950s and 60s."*
- *"Which records do I own on Blue Note?"*
- *"What's my most expensive record?"*
- *"Do I have any duplicates?"*
- *"Show me everything in my Favourites folder."*
- *"Which records are marked for upgrade?"*

**Wantlist**
- *"Summarise my wantlist."*
- *"What styles make up my wantlist?"*
- *"Which of my House wants are currently for sale?"*
- *"Show me my unscanned Techno wants."*
- *"Which 1990s records am I looking for?"*

**Buying runs**
- *"Which sellers have the most of my wants?"*
- *"Who covers the most of my House wants, VG+ minimum?"*
- *"Compare Seller 1 and Seller 2 for House and Deep House."*
- *"What does Seller 1 have that Seller 2 doesn't?"*
- *"Export the seller coverage to CSV."*

**Purchases**
- *"What did I buy from Seller 1?"*
- *"Show me my most expensive GBP purchases."*
- *"What did I buy in 2024?"*

---

*DisCat MCP User Guide — v1.0 — March 2026*
