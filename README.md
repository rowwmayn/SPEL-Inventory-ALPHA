# Sherpa Power Engineering — Product Catalog
 
A single-page HTML/JS product catalog. No backend, no build step — it's a static
`index.html` that fetches one CSV file and renders a searchable, sortable,
filterable table in the browser.
 
---
 
## 1. File structure
 
```
/
├── index.html              ← the entire site (HTML + CSS + JS in one file)
└── data/
    └── inventory.csv        ← the ONLY data source the site reads
```
 
That's it. There is no server-side code. Updating the catalog means updating
`data/inventory.csv` and re-uploading it — the page re-parses it on every load.
 
There is also a separate working file that is **not** part of the deployed
site:
 
- `Sherpa_Inventory.xlsx` — the master spreadsheet you edit by hand. You
  maintain data here, then export/download it as CSV and overwrite
  `data/inventory.csv` with that export. The `.xlsx` file itself is never
  uploaded to the site.
---
 
## 2. The update workflow (the part you'll actually do)
 
1. Open `Sherpa_Inventory.xlsx`, edit the **Inventory** sheet (add rows,
   change quantities, fix a model number, whatever).
2. Keep the header row and column order exactly as they are — the site
   matches columns by name, not position, but the names must match exactly.
3. **File → Save As / Download → CSV (Comma Separated Values)**.
4. Rename the exported file to `inventory.csv`.
5. Replace `data/inventory.csv` in the site's folder with that file.
6. Refresh the page. Done — no code changes needed.
---
 
## 3. CSV schema
 
`data/inventory.csv` must have this header row, in this order:
 
| Column             | Used for                                                        |
|--------------------|-------------------------------------------------------------------|
| `Sl No`            | Optional. If blank, the site auto-numbers rows `SKU-001, SKU-002…` in file order. If reordered, serials shift. |
| `Item Name`         | → **Category**. Title-cased on display (known acronyms like MCB, VFD, APFC, MCCB, ACB, SPD, DC, AC, PV, MPPT, TWP, MFM, MMR are forced uppercase). |
| `Brand`             | → Brand column, shown as-is (trimmed).                          |
| `Model`             | → Model column, shown as-is (trimmed).                          |
| `Power/Capacity`    | Free-text rating (e.g. `585wp`, `5.5 kw`, `30A`, `DC40`, `Accessories`). Parsed for a display rating — see §4. |
| `Qty`               | → Stock column. `0`/blank shows as "N/A" (out of stock) instead of `0`. |
| `TWP`               | "Total Watt Peak" — a pre-computed numeric rating, used **before** parsing `Power/Capacity` when it's present. Mainly used for solar panels. |
| `Product Location`  | → Location column (e.g. `Shed`, `WH1`, `RDF`). Title-cased like Category. |
| `Prices`            | → Price column, shown in Taka (৳). Blank shows "N/A". |
| `Remarks`           | **Ignored entirely by the site.** Kept in the spreadsheet only for your own reference (datasheet links, etc.) — do not expect it to appear anywhere on the page. |
 
⚠️ Any column not in this list is silently ignored by the parser. Don't rename
existing columns — the JS reads them by exact header name (`raw['Item Name']`,
`raw['Power/Capacity']`, etc.).
 
---
 
## 4. How "rating" (the Power/Capacity column on the site) is derived
 
Because the source data is messy free text, the site tries several things in
this priority order for each row:
 
1. **`TWP` field**, if it's a plain number → used directly. Unit is guessed
   as `Wp` if the category contains "panel", otherwise `W`.
2. Otherwise, regex-scan `Power/Capacity` for the first match, checked in this
   priority order: `kW → Wp → HP → kVA → kA → VA → DC<number> (treated as A) → A → W`.
3. If nothing numeric is found (e.g. `"Accessories"`, `"220V 50Hz"` with no
   power figure), the raw text is displayed as-is and the row simply isn't
   sortable/searchable by rating.
This is a best-effort heuristic, not a guarantee — multi-value fields like
`"1.5kw/2hp/220v/50hz"` will only surface the first matching rating (kW wins
over HP/voltage/Hz in that example).
 
---
 
## 5. Search & filtering behavior
 
- The category filter buttons are generated automatically from whatever
  distinct values appear in `Item Name` — no manual list to maintain.
- Free-text search matches against serial, category, brand, model,
  power/capacity text, and location, AND-ing multiple words together
  (e.g. "1.5kw pump" narrows to 1.5kW items in pump-like categories).
- A small alias map (`CATEGORY_ALIASES` in the JS) lets loose terms like
  "breaker", "pump", "inverter", "controller", "surge" match multiple related
  categories even if that exact word isn't in the category name. If you add a
  brand-new category that should be reachable via a shorthand term, add an
  entry there.
- Typing a bare number or a number+unit (e.g. `585`, `585wp`, `5.5kw`, `30a`)
  matches rows by their extracted rating.
---
 
## 6. Known limitations / things to double check after a big data update
 
- Rows with no numeric rating (pure "Accessories", voltage-only specs, etc.)
  are expected — not a bug.
- If you introduce a new category abbreviation (e.g. a new acronym-style item
  name), it will Title Case by default (`"Xyz Thing"`) unless added to the
  `ACRONYMS` set in the JS.
- Serial IDs (`SKU-XXX`) are positional when `Sl No` is blank — reordering
  rows in the spreadsheet changes which SKU number a product gets. If you
  need stable, permanent SKUs, start filling in `Sl No` for every row.
- Prices are currently all blank in the source data ("N/A" everywhere on the
  live site) — this is expected until pricing is added to the spreadsheet.
---
 
## 7. If you need AI help later
 
Paste this file plus `index.html` (and optionally a sample of
`data/inventory.csv`) into a fresh chat and say something like:
 
> "Here's my Sherpa Power Engineering product catalog site (README + index.html).
> I want to [describe the change] — can you update it?"
 
That gives a new assistant everything it needs: the file structure, the exact
CSV schema, how rating/category parsing works, and the known quirks — without
having to reverse-engineer the JS from scratch.
 