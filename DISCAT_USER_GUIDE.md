# DisCat v2.0 — User Guide
**Version:** 2.0

DisCat is a desktop app for managing your Discogs vinyl collection. It downloads your collection to a local database, tracks purchase history, enriches records with purchase metadata, and lets you find the best sellers across your entire wantlist.

---

## Contents

1. [First-Time Setup](#1-first-time-setup)
2. [Download Your Collection](#2-download-your-collection)
3. [Browse Your Collection](#3-browse-your-collection)
4. [Purchase History](#4-purchase-history)
5. [Enrich Your Collection](#5-enrich-your-collection)
6. [Sync Metadata to Discogs](#6-sync-metadata-to-discogs)
7. [Wantlist](#7-wantlist)
8. [Stats](#8-stats)
9. [MCP Server — DisCat in Claude Desktop](#9-mcp-server--discat-in-claude-desktop)
10. [Web UI (Companion Browser App)](#10-web-ui-companion-browser-app)
11. [Tips & Troubleshooting](#11-tips--troubleshooting)

---

## 1. First-Time Setup

### What you need
- A Discogs account
- A Discogs API application (free): discogs.com/settings/developers → **Create an App**
- DisCat.exe

### Connect to Discogs
1. Open DisCat.exe
2. The setup wizard opens on first run
3. Enter your **Consumer Key** and **Consumer Secret** from your Discogs app
4. Enter your **Discogs Username**
5. Choose an **Output Directory** — where DisCat keeps your data (see below). Pick a folder you back up, e.g. `C:\DisCat\data`
6. Click **Authenticate** — a browser window opens
7. Authorise DisCat on Discogs.com
8. Copy the verifier code back into DisCat
9. Click **Save** — credentials are stored in `config.env`

Your settings are saved and restored automatically on every launch.

### Where your data lives

Everything DisCat creates lives in your **Output Directory** — there's no separate location to configure for each thing:

- **`discat.db`** — your whole collection, wantlist, purchases, custom fields, market values and playlists. This single SQLite file *is* your database; back it up and you've backed up everything.
- **`covers/`** — the cached front-cover images (created when you use 📷 Fetch Covers).
- **`discat.log`** — a small diagnostic log (warnings and errors only, by default). If something misbehaves, this file is what to look at — see [Tips & Troubleshooting](#11-tips--troubleshooting). It's capped at 1 MB with a few rotated backups, so it never grows beyond a few megabytes.
- **CSV exports** — every export (collection, enriched collection, wantlist coverage, market values, manual purchase history) lands here **by default**, named automatically (e.g. `discogs_collection_2026-06-09_1430.csv`) so exporting never asks you where to save. You can send exports to a different folder with the **Export to:** field on the Export tab (see below). (Imports, by contrast, always let you pick the file to read.)

If you never set an Output Directory, DisCat falls back to the folder it's run from — which is easy to lose track of, so it's worth setting one deliberately.

### Changing the Output Directory later

You're not locked into your first choice. On the **Download Collection** tab, use the **Save to:** field — type a path or click **Browse…** — and DisCat saves the new location immediately. From then on, the database, covers and exports all use the new folder.

> **Moving existing data:** changing the Output Directory only points DisCat at the new folder — it doesn't move your current `discat.db`. To keep your existing collection, close DisCat, copy `discat.db` (and the `covers/` folder) into the new directory yourself, then reopen DisCat. Otherwise it'll start a fresh, empty database there.

---

## 2. Download Your Collection

**Tab: Download Collection**

Downloads your entire Discogs collection into the local `discat.db` database.

### Steps
1. Go to the **Download Collection** tab
2. Choose your options:
   - **Include custom field values** — fetches your existing Discogs custom field data (recommended)
   - **Use cache** — skips items unchanged since last download (much faster on repeat runs)
   - **Generate summary report** — writes a text summary to your output folder
   - **Genres & Styles** — adds genre/style tags (slightly slower)
   - **Credits & Personnel** — adds detailed credits (slower)
   - **Tracklist** — adds full tracklists (slower)
3. Click **⬇️ Download Collection**
4. Progress shows in the log — around 1 minute without metadata, ~11 minutes with full metadata

### Smart Cache
The cache detects only genuinely new or changed items. **Folder renames do not trigger a re-download.** If you rename a folder on Discogs, DisCat skips it instantly rather than re-fetching hundreds of records.

### Safe Re-downloads
Re-downloading never loses your purchase data. DisCat merges new Discogs data while preserving all enriched purchase fields (`price paid`, `seller`, `order link`, etc.).

### Stale entry cleanup
On each download, DisCat removes any records that are no longer in your Discogs collection — so the local database stays in sync if you've sold or deleted items.

### Partial download safeguard

If any folder fails to download fully (e.g. transient `429 Too Many Requests` from the Discogs API), DisCat retries that folder's page with backoff (5s → 15s → 60s). If retries are exhausted, the folder is skipped AND stale-item cleanup is also skipped for that run — so a network blip can never silently delete real records from your local DB. You'll see a yellow `⚠️ Partial download` line in the log; just re-run **Download Collection** once the API recovers.

### 🔄 Sync Custom Fields

Refreshes custom field values in the local DB without running a full collection download. Two modes: **fresh** (pulls latest values from the Discogs API — fast, collection pages only, no per-release calls) or **cache** (re-processes raw JSON already stored locally — instant, no API calls). Use **fresh** after editing custom fields on the Discogs website.

---

## 3. Browse Your Collection

**Tab: Collection → List Collection**

After downloading, switch to the **List Collection** sub-tab to browse everything in your local database without making any API calls.

### Loading and filtering
- Click **🔄 Load Collection** to load all records from the database
- Use the **Folder** dropdown to filter to a specific folder
- Use the **Search** box to live-filter by artist or title as you type
- Use the **Year**, **Label**, and **Styles** dropdowns for additional filtering
- Click **Reset View** to clear all filters and reload

### Columns
Toggle columns on/off using the checkboxes above the list. Available columns include Artist, Title, Year, Label, Catno, Country, Folder, Date Added, Price Paid, Price (Home Currency), and market value columns (Mkt Median, Avg, High, Low, Last Sold — hidden by default).

Click **↕ Reorder** to drag columns into your preferred order. Column visibility and order are saved between sessions.

Click any column header to sort ▲/▼.

### Interacting with records
- **Double-click** a row → opens Edit Record popup
- **Right-click** → context menu with **Open on Discogs**, **Edit Record**, **Copy as JSON**, and **Delete Record**

### Edit Record

Right-click → Edit Record opens a popup to edit artist, title, rating, folder, notes, and all custom fields. Changes are saved locally and synced to Discogs immediately via the API. Dropdown custom fields (e.g. Media Condition, Bought New) render as dropdowns; text fields render as text boxes.

**Note on clearing dropdown values:** the Discogs API doesn't support blanking dropdown custom fields (any value sent must match a configured option). If you blank a dropdown in Edit Record, DisCat clears it locally and shows a notice that the Discogs side is unchanged. Workaround: define a "No" (or similar non-empty "off") option for the dropdown on Discogs — picking that value will sync cleanly. Affected fields: any single-option dropdown like `Upgrade`, `Bought New`, `Box Set`.

### Change Release (Edit Record)

The Record Info block in Edit Record has a **🔀 Change Release…**
button to re-point a collection record at a different Discogs release (wrong
pressing added, or the Discogs entry was merged/split). Because Discogs has no API
to change an instance's release, DisCat **adds a new instance for the chosen
release and removes the old one**, carrying over your saved folder, rating, notes,
custom fields and purchase data. It validates the new release ID first (nothing is
changed if it's invalid), then confirms — note the record gets a **new instance ID**
and its **Date Added resets to today** (Discogs stamps that on add and it can't be
preserved). The add always happens before the delete, so a failure can't lose the
record.

### Tracklist in Edit Record

Edit Record now shows the record's tracklist in a read-only monospace block between Record Info and Purchase Details. Format: `position: [artist — ]title - duration` (e.g. `A1: Let's Go (Fast Eddie's Mix) - 5:08`). Compilations show the per-track artist with an em-dash separator; single-artist releases skip it. Duration appears only when Discogs has it (often blank for vinyl). Capped at 8 visible lines with internal scroll for long tracklists. Selectable so you can copy a track title; otherwise read-only. Skipped entirely when the record was downloaded without "Genres & Styles" enabled (no tracklist data to show).

### 🎵 Track Search tab ⚠️ *Documentation pending*

> *This feature was added 2026-07-22. Full documentation to be written.*
>
> **What it does:** A dedicated Collection sub-tab listing every track across your whole collection (position, track title, track artist, duration, plus the owning record and its year/folder). Filters live as you type — matches track title, track artist, and record artist/title. Double-click a track to open the owning record in Edit Record; 🔄 Reload Tracks refreshes after a download. There's also a companion **🎵 incl. tracks** checkbox on the List Collection search row that makes the normal grid search match track names too.

### Purchase Order — manual edit toggle

The `Purchase Order` custom field in Edit Record's My Collection section is read-only by default (so a Discogs marketplace order link can't be accidentally overwritten) but unlockable via a checkbox: **✏️ Edit manually (for non-Discogs purchases)**. Tick it and the read-only label is replaced with an editable text box pre-populated with the current value, letting you log a Bandcamp / indie shop / record store invoice reference. Save round-trips through the normal custom-field sync path: stored locally and pushed to Discogs. Untick (or never tick) and the field stays at its current value — no spurious sync events. Intended for collections that mix Discogs marketplace orders with purchases from other sources.

### Upgrade → Wantlist auto-mirror

Marking the `Upgrade` custom field as `yes` (in Edit Record or via Field Sync → Execute) automatically PUTs the release onto your Discogs wantlist. Clearing the flag (to `No`, blank, or any other value) prompts to remove the release from your wantlist. Adds are silent; removes always confirm. Decline-on-remove leaves the wantlist alone but still syncs the Upgrade clear.

**Note:** Wantlist add/remove only fires when the Upgrade field POST itself succeeds on Discogs — so if you blank an Upgrade dropdown (which Discogs rejects, see above), the wantlist is left alone too. To actually remove a want via this flow, set the dropdown to a real `No` option rather than blanking it. The `Sync Custom Fields → fresh` path does NOT trigger wantlist changes (use the Discogs website directly if you flipped Upgrade there).

### Copy as JSON

Right-click → 📋 Copy as JSON copies the full record details to your clipboard as pretty-printed JSON, ready to paste into Gemini, ChatGPT, Claude, or any other LLM for ad-hoc Q&A about the record. The payload includes every collection column (artist, title, year, label, catno, folder, rating, notes, dates), unpacked metadata (genres, styles, country, tracklist, credits, lowest price, num for sale, community have/want counts), all custom fields (Upgrade, Bought New, Price Paid, etc.), purchase details (price, currency, date, seller, order, media/sleeve condition), and market stats if cached. A confirmation popup shows the byte count so you can see how big the paste will be before sending.

### Delete Record

Right-click → 🗑️ Delete Record removes the item from both DisCat and your Discogs collection after confirmation. Keyed on `instance_id` so if you own two copies of the same release, only the right-clicked one is deleted.

### 🔍 Check Missing

Compares your purchase history against your local collection and lists any purchases not found in the collection. Shows artist, title, purchase date, price, media/sleeve condition and order number (double-click to open the order on Discogs). Right-click any row to mark it as **Not Received** (hides it from the list persistently). Select items and click **➕ Add to Discogs Collection** to add them to Discogs and import them into DisCat automatically, including backfilling purchase data and custom fields.

**Cancelled orders are excluded** — purchases with a status starting with 'Cancelled' never surface here (no physical item arrived, so offering them as 'missing' would create phantom collection entries).

### Market Values

Scrapes Discogs sales history for collection releases and stores median, average, high, low prices and last sold date. Requires a logged-in Chrome session (same as wantlist scanning). Results appear in the market value columns in the collection list (rendered in your base currency from Settings). Use **⏹ Stop** to cancel.

- **📈 Fetch Market Values — All** — scrapes every release in the collection. **Skips releases marked `(M)` (manual entries)** so hand-entered values are sticky. Status bar shows the skip count.
- **📈 Fetch Market Values — Selected** — scrapes only the rows you've selected in the collection list (Ctrl-click or Shift-click for multi-select). Will overwrite manual entries — explicit selection counts as deliberate.
- **Listings fallback** — if a release has no Discogs sales history (typically new pressings), DisCat scrapes the current marketplace listings page instead and computes median/avg/high/low across asking prices. These values display with an `(L)` suffix to flag that they're derived from listings, not completed sales (so expect a slight high-bias).
- **Manual override (Edit Record)** — open Edit Record and scroll to the "Market Values (manual override)" section. Enter Median / Average / High / Low / Currency / Last Sold by hand. Saved values appear with an `(M)` suffix in the collection list's Mkt columns.
- **📤 Export Market Values CSV / 📥 Import Market Values CSV** (Export tab) — round-trip bulk editing. Export writes one row per release including blanks for unfetched ones; tick "Only releases without market values" to limit to the gaps. Edit median/avg/high/low/currency in Excel, then Import — matched on `release_id`; rows where every value is blank are skipped (so a no-op round-trip won't blank existing data); imported rows are always tagged `(M)`. Currency accepts either symbol (`A$`, `£`) or ISO code (`AUD`, `GBP`) — both end up stored as ISO.

### Multiple copies of the same release

DisCat now treats each copy of a release as an independent instance. If you've bought the same record twice from two different sellers, each purchase is paired to its own collection instance (oldest purchase → earliest instance). Check Missing correctly surfaces any unmatched purchases — including a second copy bought from a different seller. Edit Record always edits the specific instance you right-clicked, never its siblings. Export CSVs emit one row per instance.

### Suffix legend

A small gray reference line above the collection table reminds you what the three short suffixes mean when they appear in price columns:
- **(e)** — the Price Paid value is an estimate (entered via the bulk CSV with the `estimated` column set to TRUE), not a known price from a receipt or order.
- **(M)** — Market values were entered manually (via Edit Record's manual override or the Market Values CSV import), not scraped from Discogs.
- **(L)** — Market values were derived from current marketplace listings rather than completed sales — typically for new pressings with no sales history yet. Expect a slight high-bias compared with sold-price medians.

### Catalog Cover Photos

The Collection tab's **📷 Fetch Covers — All** button
downloads each record's front-cover image into a local cache
(`<output_dir>/covers/`), skipping any already cached. The companion Web
UI then displays the thumbnails (with a vinyl-disc fallback for any not
yet fetched). Requires a one-time DB migration to schema v9 (happens
automatically the first time DisCat opens after the update).

### Grid data sources — curated custom fields

The List Collection grid now treats your Discogs
custom fields as the source of truth for **Year, Label, Style,
Country, Catalog #, Format** (Catalog # and Format are new
columns, default-visible). Only **Artist, Title, Folder, Date
Added** come from Discogs basic_information; **Price Paid** stays
as the linked purchase-order price with the `Price Paid` custom
field as a fallback; **Upgrade** stays as a custom field. A blank
cell now means "this custom field is unpopulated for this
record" — i.e. a real backfill target. Pair this with the new
Custom Field Audit panel (in the Organise Fields tab) to see
per-field coverage at a glance.

### Discogs Styles column

A new default-visible **Discogs Styles** column
sits between Title and Folder on the List Collection grid, showing
the full multi-style list Discogs has on each release (sourced
from `raw_json.detailed_metadata.styles`). Distinct from the
curated single-value **Styles** column (sourced from the `Style`
custom field) — keep both visible when sorting records into
style-based folders so you can see every style tag Discogs has
alongside your own canonical pick. Blank if the record was
downloaded without the "Genres & Styles" option.

---

## 4. Purchase History

### Sales & Purchases tab

**Purchase History** is a sub-tab under the **Sales & Purchases** parent tab. A companion **Sales History** sub-tab covers seller-role orders and is where the **🔌 Fetch via API** button lives (the Discogs `/marketplace/orders` endpoint is seller-scoped, so it returns nothing for buyer-only accounts — that's why it sits with Sales rather than Purchases).

**Note:** Sales History is still in progress — its list view and aggregate stats aren't finished yet.

**Tab: Sales & Purchases → Purchase History**

Imports your Discogs purchase orders so they can be matched to your collection.

### Option A — Scrape from Website (recommended)
1. Click **🌐 Scrape from Website**
2. Chrome opens — log into Discogs
3. DisCat automatically scrapes all your order IDs
4. Then fetches full order details via API
5. Orders saved to `discat.db`

### Option B — CSV Import
1. Prepare a CSV file with an `order_id` column
2. Click **📂 Import Order IDs CSV**
3. Select your file — DisCat fetches details via API and saves them

### Adding a single order manually
Click **➕ Add Order** to enter a single Discogs order ID — useful if you know a specific order is missing.

### Viewing Purchase History
The purchase table shows up to 9 columns. Toggle columns using the checkboxes at the top:

| Column | Default |
|--------|---------|
| Artist | ✅ |
| Title | ✅ |
| Price | ✅ |
| Currency | ✅ |
| Date | ✅ |
| Seller | optional |
| Media Condition | optional |
| Sleeve Condition | optional |
| Order ID | optional |

Click any column header to sort. Click again to reverse.

---

## 5. Enrich Your Collection

**Tab: Sales & Purchases → Purchase History → Enrich Collection**

Matches your purchases to your collection and embeds the purchase data inside the database.

### Steps
1. Make sure you have downloaded your collection (Step 2) and imported purchases (Step 3)
2. Click **🔗 Enrich Collection**
3. DisCat matches purchases to collection items by release ID
4. For each match it stores:
   - Purchase price
   - Purchase date
   - Purchase order ID
   - Seller name
   - Media condition
   - Sleeve condition
5. An enriched CSV is exported to your output folder

### Result
Every matched record in your database now carries full purchase provenance. This data is preserved through all future collection re-downloads.

**Typical match rate: ~93%** (some releases may have mismatched IDs)

---

## 6. Sync Metadata to Discogs

**Tab: Organise Fields**

Writes your purchase metadata back to Discogs as custom fields — so it appears on your Discogs collection page.

### One-time setup on Discogs.com
1. Go to discogs.com → Settings → Collection → Custom Fields
2. Create text fields for whichever data you want to sync:
   - `Price Paid`
   - `Purchase Date`
   - `Order Link`
   - `Seller`
   - `Media Condition`
   - `Sleeve Condition`

### Syncing each field in DisCat
1. Go to the **Organise Fields** tab
2. Select the field from the **Custom field** dropdown (auto-populated from your DB; click ↻ to refresh after a Sync Custom Fields run)
3. In **Sync from metadata**, choose the matching source (e.g. `Price Paid`)
4. Click **👁️ Preview Sync** — see every record that will be written before committing
5. Click **✅ Execute Sync** — values are written to Discogs via API

Repeat for each field you want to sync.

### Organise Folders

Moves records between Discogs folders based on style/genre matching rules. **Source folder** and **Target folder** dropdowns auto-populate from the local DB on startup (click ↻ to refresh). Match modes: Style (contains any), Style (contains all), Genre (contains all). A "within first N styles" limiter restricts matching to the most prominent styles on a release.

> **⚠️ *Documentation pending* — case/separator-insensitive matching (added 2026-06-10).** Style/genre matching now ignores case and separators: `synth pop`, `Synth-pop` and `Synthpop` all match a `Synth-pop` tag, and matching is substring-based (`progressive` catches `Progressive House`). The "First Style == folder" mode is likewise case/separator-insensitive. Full write-up to follow.

### Custom Field Audit popout + Setup dialog

The **📋 Custom Field Audit…** button on the
Organise Fields tab opens a modeless popout window. A status line
shows setup completeness (e.g. `⚠ Setup: 1 required field(s)
missing on Discogs — Country`). A treeview lists every canonical
DisCat custom field with **Populated / Missing / Coverage %**.
Fields not yet defined on Discogs show `✗ not on Discogs`.
**Double-click a row** to load that field into the Sync
Configuration combo on the underlying tab — then pick a metadata
source and run Preview / Execute to bulk-backfill.

**🔧 Check Setup** opens a modal dialog listing every missing
canonical field with its required `Type / Lines / Options /
Purpose`, plus a **🔗 Open Discogs Settings** button that deep-links
to your Collection Settings page (the Discogs API has no public
endpoint for creating custom fields, so they must be added by
hand). **↻ Re-check** re-runs the comparison without closing.

**Auto-show after Sync Custom Fields** (checkbox in the dialog):
on by default; persisted as `AUDIT_SETUP_AUTO_SHOW` in
`config.env`. While enabled, the dialog opens automatically after
any Sync Custom Fields run when canonical fields are still missing
from Discogs.

### Order Links
The `Purchase Order` sync option generates a clickable URL from your order ID:
```
Order 62035-956 → https://www.discogs.com/sell/order/62035-956
```
One click from your Discogs collection page takes you straight to the original listing, photos, and messages.

### Use Local Value sync source

A sync source that pushes the value currently stored in your **local custom_fields** table up to Discogs unchanged. Use it after a bulk CSV import (e.g. Price Paid) when you want the imported values to land on Discogs without retyping them via Edit Record. Pick the target field name (e.g. `Price Paid`), select **Use Local Value** as the source, Preview → Execute. Works for any custom field — same flow for `Purchase Order` or anything else.

---

## 7. Wantlist

**Tab: Wantlist** (two sub-tabs: **Coverage** and **Releases**)

### Sync your wantlist first
Click **📥 Sync Wantlist** to pull your current wantlist from Discogs into the local database. Always do this before scanning if your wantlist has changed.

### Coverage sub-tab
Finds which sellers on the Discogs marketplace stock the most items from your wantlist — ranked by coverage and price.

| Button | Chrome needed? | What it does |
|--------|---------------|--------------|
| **📂 Load from Cache** | No | Instantly loads previously scanned results from the database |
| **🔍 Scan Missing** | Yes | Scrapes only releases not yet in cache (use for new wantlist additions) |
| **🔄 Refresh All** | Yes | Force re-scrapes all releases regardless of cache age |

**When Chrome opens:** log into Discogs when prompted. DisCat waits up to 2 minutes for login, then begins scanning automatically. Chrome stays open in the background — do not close it until the scan completes.

Sellers are ranked by coverage (most of your wants first), then by total price. Columns: Seller, Coverage, Total Price, Currency, Rating, Country, Availability.

**Filter by condition** using the Min Media and Min Sleeve dropdowns. **Hide unavailable** checkbox filters out region-restricted sellers (those who can't ship to you).

**Single-click a seller row** → shows that seller's listings in the bottom pane. **Double-click** → opens `discogs.com/seller/{name}/mywants` in your browser.

**⧉ Pop Out** opens the coverage list in a separate window with a split pane: top half shows sellers, bottom half shows all listings for the selected seller.

**Seller search box** — type to filter the seller list live.

**Cache TTL** — set how long (in hours) cached listings are considered fresh before Scan Missing will re-scrape them.

### Releases sub-tab
Browse every marketplace listing for a single wantlist item.

1. Select a release from the dropdown (shows Artist — Title)
2. Click **🔍 Search** — all cached listings are shown: Price, Availability, Media, Sleeve, Comments, Ships From, Seller, Rating
3. Use the **real-time search box** to filter listings by any text as you type
4. Set **Min Media** / **Min Sleeve** to filter by condition

**Double-click a listing** → opens that specific listing on Discogs.

**🔄 Force Rescrape** — re-scrapes this release's listings from scratch, ignoring cache. Use when the cached data looks stale for a specific record.

**⚠️ Availability column:** ✓ = ships to your region, ⚠ = region-restricted. Listings scraped before this feature was added may show ⚠ incorrectly — use Force Rescrape to refresh.

### Fair Value summary

Shows a live AVG + MEDIAN of the currently-visible listings, converted to your base currency, sitting next to the listing count. Updates as Min Media / Min Sleeve / Ships From / Hide unavailable change — so the number always reflects "what this release costs at the conditions I'd actually accept". Listings priced in a currency missing from your stored exchange rates are excluded with a count.

### Shared Chrome session
If you already logged in on the Coverage tab, the same Chrome session is reused in Releases — no second login needed.

---

## 8. Stats

**Tab: Stats**

Shows a dashboard of your collection across several panels.

### Collection summary
Total items, folder breakdown, format split, genre and style breakdown.

### Purchase summary
Total spend, number of purchases matched to collection items.

### Wantlist summary
Total wantlist items, how many have been scanned, scan state breakdown (Pending / For Sale / Not Listed).

### Collection Value

Shows estimated total collection value based on Discogs median sale prices, adjusted for each record's condition grade. Displays total value, average per record, market data coverage %, and a Top Records by Value list. Use the spinbox next to the list to choose how many records to show (10–50 in increments of 10; default 10). Requires market values to be fetched first (📈 Fetch Market Values — All or — Selected on the Collection tab). Condition grades apply multipliers (Mint = 1.5×, NM = 1.25×, VG+ = 1.0×, VG = 0.6×, etc.).

**Format filter** — three checkboxes under the Top Records list (`12"/LP`, `Album`, `Box Set`) restrict the list using **mutually-exclusive tiers** with precedence `Box Set > Album > 12"/LP`. Each release is classified into exactly one tier (or none) by the highest-priority tag it carries: a 12" LP tagged Album appears only under Album; a deluxe edition tagged Album+Box Set appears only under Box Set. Ticked checkboxes union the matching tiers; no boxes = no filter. Filter applies to the Top N list only — the Total Value, Avg per Record, and Market Data coverage stats stay calculated across the whole collection.

### Export

**Tab: Export**

All export buttons live on the **Export** tab.

**Export location.** By default exports are saved to your Output Directory (alongside `discat.db`). To send them somewhere else — a Dropbox folder, a USB drive, a dedicated `exports` folder — use the **Export to:** field at the top of the tab: type a path or click **Browse…**, and the choice is remembered between sessions. Click **Use Output Dir** to clear it and return to the default. A status line shows exactly where exports will land. This affects exports only; imports always prompt you to pick the file to read.

**Collection CSV columns:** artist, title, label, catno, format, year, folder, rating, notes, date_added, last_updated, purchase_price, purchase_currency, purchase_date, purchase_seller, media_condition, sleeve_condition, market_median, market_avg, market_high, market_low, market_last_sold, market_currency.

### 💷 Price Paid + Purchase Order (manual)

Bulk export/edit/import flow for filling in the `Price Paid` and `Purchase Order` custom fields for non-Discogs purchases (record store, eBay, etc.). Mirrors the Market Values CSV pattern but operates **per-instance** (so two copies of a release are separate rows) and writes to the `custom_fields` table only — does not touch the raw_json `purchase_*` fields used for Discogs marketplace orders.

- **📤 Export Price Paid CSV** — one row per instance with columns `instance_id, release_id, artist, title, year, Label, Catalog #, Price Paid, estimated, Purchase Order`. The `year`, `Label`, `Catalog #` columns are read-only reference data (useful for estimating prices); they are not written back on import. Tick **"Only instances with empty Price Paid"** to limit the export to rows that need filling.
- **`estimated` column** — set to `TRUE` (or `yes`/`1`/`y`) to mark the price as estimated; on import this appends ` (e)` to the value. Round-trip safe: export strips the `(e)` into the column for clean editing.
- **📥 Import Price Paid CSV** — matches on `instance_id`. Currency-normalises Price Paid to `ISO N.NN` format (`$15` → `AUD 15.00`, `£12.50` → `GBP 12.50`, bare `15` → `AUD 15.00` using your base currency, etc.). Shows a **confirmation dialog** with counts (writes / estimated / skipped / bad rows) before any DB write — say No to abort. Skips rows where both Price Paid and Purchase Order are blank.
- **Pushing to Discogs** — the import only writes to your local DB. To push the values up to Discogs, use **Collection → Organise Fields** (sub-tab inside the Collection tab), choose target field `Price Paid`, sync source `Use Local Value`, then Preview Sync → Execute Sync. Same pattern for `Purchase Order`.

### 📊 Auto-Estimate Empty Prices

Auto-fills the `Price Paid` custom field for collection items that don't have a Price Paid value yet, using a tier-based estimator keyed on (release year + format + origin) — calibrated for records bought new in Australia between 1988 and 2000.

- **Selection** — only items with NO existing Price Paid are touched. Real values, manually-entered values, and previously-estimated `(e)` values are all left alone. To re-estimate a row, clear its Price Paid first.
- **Preview window** — opens a sortable table showing artist / title / year / format / origin / estimated AUD per record, plus the total estimated spend. Cancel or Confirm before any DB write.
- **Tier table** — three formats (12" single / EP / LP), two origins (IMPORT / DOMESTIC by `country` field; falls back to label name + catno prefix when country missing), five year bands (early ≤1992 / mid 93-96 / late 97-2000 / post 01-09 / modern 2010+). Modern LPs with premium-pressing signals (180g, coloured vinyl, picture disc) get a $13-15 uplift to match the $35-45 retail of those reissues.
- **`(e)` suffix** — every estimate is written with the `(e)` marker so you can tell estimates apart from real prices in the List Collection treeview (legend below the column toggles explains this).
- **Auto-push to Discogs** — after the local DB write, asks "Push to Discogs?". Yes runs a background API push with a progress modal, writing each Price Paid value directly to the matching Discogs collection-instance custom field. No leaves the values local-only; you can push later via Collection → Organise Fields → `Use Local Value`.
- **Out of scope** — CDs, cassettes, and 7"-only releases return None and are skipped (the tier table is calibrated for 12"/LP vinyl).

---

## 9. MCP Server — DisCat in Claude Desktop

The DisCat MCP server lets you query your collection and wantlist data conversationally through Claude Desktop, using natural language instead of the GUI.

### What you can ask Claude
- *"Which seller covers the most of my wantlist?"*
- *"Show me all sellers shipping from the UK with VG+ or better media"*
- *"Sync my wantlist and then tell me the top 10 sellers by coverage"*
- *"Export a Markdown report of seller coverage"*
- *"What listings are available for [release name]?"*

### Setup

`DisCatMCP.exe` is self-contained — like `DisCat.exe`, it bundles everything it needs, so there's nothing to install first.

#### Step 1 — Configure Claude Desktop
Edit `%APPDATA%\Claude\claude_desktop_config.json` (create it if it doesn't exist):
```json
{
  "mcpServers": {
    "discat": {
      "command": "C:\\DisCat\\exe\\DisCatMCP.exe",
      "args": []
    }
  }
}
```
Adjust the path to wherever `DisCatMCP.exe` lives alongside `DisCat.exe`.

#### Step 2 — Restart Claude Desktop
The DisCat tools will appear in Claude's tool list automatically.

---

### Available MCP Tools

#### `wantlist_list`
Lists all items in your local wantlist database.
- Optional: specify which fields to return (e.g. `release_id`, `artist`, `title`)
- No network connection required

#### `wantlist_sync`
Syncs your wantlist from Discogs API into the local database.
- Uses your stored OAuth credentials — no Chrome needed
- Run this if your wantlist has changed since the last DisCat sync

#### `marketplace_listings`
Returns cached marketplace listings for a single release.
- Parameters: `release_id`, optional `min_media`, `min_sleeve`
- Returns a `no_cache` message if the release hasn't been scanned yet

#### `scan_wantlist`
Scrapes fresh marketplace listings from Discogs for your wantlist items.
- **Opens Chrome** — log into Discogs when prompted (up to 2 minutes)
- `force_refresh: false` (default) — skips already-cached releases
- `force_refresh: true` — re-scrapes everything regardless of cache
- Chrome stays open for the duration of the session

#### `seller_coverage`
Aggregates cached marketplace data to rank sellers.
- Parameters: optional `min_media`, `min_sleeve`, `top_n` (default 20)
- Returns sellers ranked by coverage count, then total price
- Runs entirely from local cache — instant, no network needed

#### `report_export`
Exports the seller coverage report to a file.
- Parameters: `format` (`csv`, `json`, or `markdown`), optional `path`
- Default path: your configured output directory

---

### Typical MCP workflow

```
You:    Sync my wantlist from Discogs
Claude: [calls wantlist_sync] → Synced 460 items

You:    Scan for any releases not yet in cache
Claude: [calls scan_wantlist] → Chrome opens, you log in
        Scanned 48 missing releases, found 312 new listings

You:    Which sellers cover the most of my wantlist,
        minimum VG+ media, shipping from UK or Europe?
Claude: [calls seller_coverage with min_media=VG+]
        Here are the top sellers...

You:    Export that as a Markdown report
Claude: [calls report_export with format=markdown]
        Report saved to C:\DisCat\data\wantlist_coverage_report.md
```

---

### Important notes
- The MCP server reads from the **same `discat.db`** as the DisCat GUI — you can run both simultaneously
- OAuth tokens are stored locally in `config.env` and are never sent to Claude
- The MCP server is a separate process from the DisCat GUI — closing the GUI does not stop the MCP server
- `DisCatMCP.exe` must be in the same folder as `config.env` so it can find your credentials and database

---

## 10. Web UI (Companion Browser App)

A localhost browser view of your collection, wantlist, stats, market values, purchases and playlists. Start it from **Settings → 🌐 Web UI → Start** (or run `DisCatWeb.exe`), then click **🌐 Open in Browser** (or visit `http://127.0.0.1:8765/`). It runs only on this PC — nothing is published. Everything except playlists is read-only; playlist building is the one writable feature (see below).

The interface is a dark, flat design with an icon-only sidebar rail, near-black surfaces, record-cover thumbnails in the Collection grid and a cover hero on the release detail panel. **Double-click any Collection row** to open its detail panel. The views:

- **Collection** — sortable grid with cover thumbnails, a **Rating** column (`4 ★`), live search and add-to-playlist.
- **Release detail** — editorial layout: blurred cover backdrop, large title, key stats in monospace.
- **Stats** — collection breakdowns with a hero header and amber bar charts.
- **Market** — per-release values shown by release name; each row **expands** to its cached marketplace listings.
- **Wantlist** — searchable by artist/title, with a compact stat strip.
- **Purchases** — your order history in a consistent toolbar layout.

### DisCat Mobile (phone PWA) ⚠️ *Documentation pending*

> *This feature was added 2026-06-13. Full documentation to be written.*
>
> **What it does:** A simple phone app (served at `/m` by the web companion) for
> "is this record in my collection?" while record shopping — searches Discogs
> directly, checks an offline-cached copy of your collection (works with no
> signal), and shows your details, a master-release link and YouTube videos.
> Reach it on your phone via your PC's LAN IP (`DISCAT_WEB_HOST=0.0.0.0` in
> `config.env`); install as a real app needs HTTPS (Tailscale). See
> `MOBILE_APP_PLAN.md`.

### Schema alignment readout (Settings → 🌐 Web UI)

DisCat, the Web UI and your database all need to agree on the database
*schema version* — when you update one EXE but not the other, they can drift.
The **Settings → 🌐 Web UI** panel shows a one-line readout so you can see at
a glance whether everything matches:

```
Schema —  discat.db v14   ·   DisCat v14   ·   Web v14
```

- **`discat.db`** — the version your database file is currently at.
- **`DisCat`** — the version this build of the desktop app expects.
- **`Web`** — the version the *running* `DisCatWeb.exe` build expects
  (shows `(stopped)` when the Web UI isn't running).

The line is colour-coded, with a hint underneath telling you exactly what to
do:

- **Green** — all aligned ✅ (or DB & DisCat aligned with the Web UI stopped).
- **Red** — your DB is *behind* DisCat. Harmless: DisCat migrates the
  database automatically on launch, so just reopen it.
- **Orange** — a mismatch involving the Web UI. Most commonly the Web build
  is older than the database ("rebuilt DisCat.exe but DisCatWeb.exe is
  stale") — rebuild/update `DisCatWeb.exe`. If the Web build is *newer* than
  the DB, reopen DisCat first so it migrates, then reload the browser page.
- **Gray** — no `discat.db` yet; run a Download in the Collection tab first.

The readout refreshes every few seconds. Checking the versions never
modifies your database — the DB is read in read-only mode, and the Web
version comes from the running server itself.

### Playlists in the Web UI

Playlists are the Web UI's one writable feature. On the Collection page, tick one
or more records (checkbox column) and a sticky action bar appears with **Add to
playlist** → pick an existing playlist or create a new one. Manage playlists on the
dedicated **📋 Playlists** page (master/detail layout, view/edit modes).

Each playlist item is a **track**, not a whole record, so the same 12" can appear in
a playlist twice for different tracks (A1 then later B2). Every track carries a
**Camelot key** (24-option dropdown, e.g. `8A — Am`, shown as a wheel-coloured chip)
and an **energy** rating 1–10 (a warm green→amber→hot bar). Key and energy are stored
per track and shared across all playlists. Each item also has a **playlist-only
rating** (1–5 stars) that's local to the playlist and never synced to Discogs, so a
re-download won't overwrite it.

Columns are **sortable** (Key sorts musically around the Camelot wheel) and
**resizable**; **Save as set order** commits the current sort as the stored order,
and **Reset view** restores the defaults. **Export CSV** produces a set sheet
(Position, Artist, Track #, Track Title, Key, Energy, Album, Year, Label, Catalog #,
Format, Folder, Rating, Notes, Discogs URL).

Keys and energy are entered manually today; a future Mixed In Key import will be able
to populate them automatically.

### Web UI Settings page

New **⚙️ Settings** page in the Web UI (sidebar
bottom). Two display preferences, both persisted per-browser via
localStorage: **Cover thumbnails** (Small 32 / Medium 48 / Large 64
px — applied to the Collection and Playlists grids) and **Font
size** (Small 12 / Medium 14 / Large 16 px — applied to the root
`<html>` so it scales the whole app). Doesn't sync with the
desktop GUI; web-only.

### Web UI design refresh — "Workbench × Booth"

Foundation aesthetic pass on the web companion.
**IBM Plex Sans + Plex Sans Condensed + JetBrains Mono** replace
system-ui (mono numerals enable tabular figures so columns don't
jitter on sort). **Amber accent** (#f59e0b) replaces blue throughout;
a **full Camelot wheel colour system** is seeded for the next pass
to consume on key/energy displays. **Custom vinyl-tool SVG icons**
replace emoji in the sidebar (tonearm icon for Settings, vinyl-record
for the brand mark — which slowly rotates). A subtle **film grain
overlay** and a **warm radial atmosphere** behind the topbar add
depth without competing with content. Per-view layout refreshes
(Collection row hierarchy, Playlists row hierarchy, ReleaseDetail
editorial) are scheduled for the next session.

### Right-click Cut / Copy / Paste

Right-click any text field or log window for Cut / Copy / Paste / Select All. Cut/Paste are greyed out on read-only fields and logs.

### Multi-monitor dialog placement

Popups and dialogs now open over the DisCat window instead of the primary monitor.

### 🔄 Refresh Record Info button (Edit Record popup)

A button in the fixed top strip of the Edit
Record popup that re-fetches the Discogs release metadata for the
current record. Updates artist, title, year, label, format,
country, genres, styles, and the raw `detailed_metadata` block
in one shot. Your folder, rating, custom fields and purchase data
are not touched. Useful when you've fixed metadata on Discogs
and want it reflected locally without re-downloading the whole
collection. The List Collection grid auto-refreshes on success.

### Edit Record popup — top action strip

Save, Cancel and Refresh Record Info now live in
a fixed strip at the top of the Edit Record popup (instead of
Save/Cancel at the bottom). The strip stays visible while you
scroll the form below. Refresh Record Info sits on the left with
a short hint explaining what it does; Save and Cancel sit on the
right.

### Notes field — auto-truncated to fit Discogs (255 chars)

The Notes custom field on Discogs is a
single-line textarea capped at 255 characters and won't accept
newlines. If you enter a longer Notes value (or any single-line
textarea custom field), DisCat now collapses newlines to spaces
and trims the value at the nearest word boundary before sending
it — the same cleaned value is also written to the local DB so
the two stores stay byte-equal and Sync Custom Fields doesn't
see a phantom diff every run. The save warning popup tells you
exactly what got cut (e.g. `Notes: 482 → 247 chars`).

If you need longer per-copy notes, raise it as a feature request
for a separate local-only Notes field — already on the backlog.

### List Collection — auto-load + Reload Collection

The List Collection sub-tab now auto-loads the
grid the first time you switch to it in a session. The button
formerly labelled **Load Collection** is now **🔄 Reload
Collection** — use it to force a manual refresh from the local DB
(e.g. after external edits).

### List Collection — column visibility + reorder now sticky

Two bugs fixed on the Columns checkbox row and
the ↕ Reorder Columns dialog:
- **Reorder survives toggling visibility.** Previously, ticking
  any checkbox would reset the column order back to default.
  Now your manual ordering is preserved when you hide or show
  columns.
- **Deselections stick across sessions.** Previously, default-on
  columns you'd deselected would come back the next time you
  opened DisCat. Now they stay hidden until you tick them again.

---

## 11. Tips & Troubleshooting

### The ℹ️ Help tab — these guides, inside the app

The last tab in DisCat is **ℹ️ Help**: an About panel plus a **📚 User
Guides** section with both guides (this User Guide and the MCP Guide). Each
guide has two buttons:

- **📂 Open** — opens the copy bundled inside the EXE with your default
  Markdown/text viewer. Works completely offline; it matches the version of
  DisCat you're running.
- **↗ GitHub** — opens the same guide rendered nicely in your browser (on
  the public `discat-help` repository — no GitHub account needed). This is
  always the latest published version.

So if you're stuck without this file handy: the answer is on the ℹ️ Help
tab.

### Something went wrong and the app didn't say why — check the log file

DisCat keeps a diagnostic log, **`discat.log`**, in the same folder as
`discat.db` (your Output Directory). It records warnings, errors and — most
usefully — anything that crashed silently behind the scenes, like a button
click that appeared to do nothing. If you're reporting a problem, attach
this file: it usually contains the actual reason.

A few things worth knowing:

- **It's quiet by default.** A normal session writes only a handful of lines
  (app start/exit plus any warnings). Nothing you do in the app is "tracked" —
  it only records problems.
- **More detail when troubleshooting:** add `LOG_LEVEL=DEBUG` to `config.env`
  (the file next to `DisCat.exe`) and restart DisCat. Remove the line (or set
  it back to `INFO`) when you're done. `INFO` is the default, so setting it
  explicitly changes nothing.
- **Less detail:** `LOG_LEVEL=ERROR` records only outright failures. There is
  no full off switch — the point of the log is that the evidence already
  exists when something goes wrong.
- **It can't fill your disk.** The file is capped at 1 MB and rotates through
  3 backups (`discat.log.1`, `.2`, `.3`) — about 4 MB worst case, ever.

---

### I edited a record on Discogs but DisCat still shows the old data

If you update a record on the Discogs website (e.g. change a condition grade, add a note, or update a custom field), you need to re-download your collection — and you must do it **with "Use cache" unchecked**.

**Why:** "Use cache" reads from the JSON saved during your last download. It skips the Discogs API entirely, so any web edits made since then won't come through.

To get the updated data: run **⬇️ Download Collection** with **Use cache** unchecked. This fetches fresh data from Discogs.

**Tip:** To avoid a full re-download, make collection edits directly in DisCat instead — right-click any record in the Collection tab and choose **Edit Record**. Changes are saved to the local database and synced to Discogs immediately via the API.

---

### Collection download is slow
Uncheck **Credits & Personnel** and **Tracklist** — these add one API call per record. Without them, a 656-item collection downloads in about 1 minute.

### My purchase data disappeared after re-downloading
It shouldn't — DisCat uses smart merge to preserve enriched data. If it did happen, check that you're not running an older version of the EXE. Run **🔗 Enrich Collection** again to re-apply purchase data.

### Wantlist scan shows 0 sellers with condition filters
Unknown conditions (blank or unrecognised values) are passed through rather than excluded. If you see 0 results, try relaxing the condition filter. Use **🗑️ Clear Cache** then **🔍 Scan Missing** to get fresh data with the latest parser.

### Country and Rating columns are empty
This means old cached data was scraped before the parser fix. Click **🗑️ Clear Cache** then run **🔄 Refresh All** to re-scrape with the corrected parser.

### Chrome closes during a scan
If Chrome is accidentally closed mid-scan, DisCat will error on the next scrape attempt. Click **🔍 Scan Missing** again — it will reopen Chrome and continue from where it left off (cached releases are skipped).

### MCP server not appearing in Claude Desktop
- Check the path in `claude_desktop_config.json` — backslashes must be doubled (`\\`)
- Make sure `DisCatMCP.exe` is in the same folder as `config.env`
- Restart Claude Desktop completely after editing the config
- To test manually: open CMD and run `DisCatMCP.exe` — if it starts without error and waits, it's working (press Ctrl+C to exit)

### MCP scan_wantlist login timeout
The MCP server waits 120 seconds for Discogs login. If you see a timeout error, call `scan_wantlist` again — Chrome will reopen for a fresh login attempt.

### API rate limit errors
DisCat automatically handles Discogs rate limits (60 requests/minute for OAuth). If you see rate limit errors, wait a minute and try again. The scraper includes delays between pages automatically.

---

*DisCat v2.0 — Your collection, fully documented, completely safe, ready to sync.* 🐱
