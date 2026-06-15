# Sherpa Power Engineering — Product Price Catalog

A static, no-backend price catalog. Built as a single `index.html` (HTML+CSS+JS combined) that reads from CSV files in `/data`. Designed to be hosted on GitHub Pages.

## Directory structure

```
sherpa-pricelist/
├── index.html                    # the entire website (HTML + CSS + JS)
├── README.md                     # this file
└── data/
    ├── solar_panels.csv
    ├── charge_controllers.csv
    ├── circuit_protection.csv    # MCB, MCCB, isolators, fuses
    └── spd_surge_protectors.csv
```

To add a **new category**, create a new CSV in `/data` with the same columns below, then add its filename to the `CSV_FILES` array near the top of the `<script>` section in `index.html`.

## CSV column schema (fixed across all files)

| Column           | Meaning                                                                 |
|------------------|--------------------------------------------------------------------------|
| `serial_id`      | Custom internal serial — see numbering scheme below                     |
| `category`       | Product category (used for grouping/filtering on the site)              |
| `brand`          | Manufacturer / brand                                                     |
| `model`          | Model number / name                                                      |
| `specifications` | Key specs, pipe-separated (`|`) — rendered as a spec list on the card    |
| `unit`           | Sales unit: `Wp`, `Pcs`, etc.                                            |
| `price_tk`       | Price in BDT (Taka). Leave blank if not applicable.                      |
| `price_usd`      | Price in USD. Leave blank if not applicable. (Tk takes priority if both present) |
| `stock_qty`      | Current stock count. `0` or blank shows "stock: N/A" on the site.       |
| `notes`          | Free text — warranty info, remarks, etc.                                  |

Keep these 10 columns in every CSV, in this order. Empty cells are fine — just leave them blank (e.g. `,,`).

## Serial numbering scheme (recommended for future inventory system)

Format: **`[Category Letter]-[3-digit running number]`**

| Letter | Category                              |
|--------|----------------------------------------|
| `P`    | Solar Panels                            |
| `C`    | Charge Controllers (and later Inverters → `I`, Batteries → `B` — see note) |
| `B`    | Circuit Breakers / MCB / MCCB / Fuses / Isolators |
| `S`    | Surge Protection Devices (SPD)          |

> Note: `B` is currently used for both "Breaker" and reserved for "Battery" in future — if/when batteries are added, consider using `BT` for batteries and keep `B` for breakers, or reassign one to a new letter to avoid collisions.

When adding a new product:
1. Pick the category letter.
2. Find the highest existing number for that letter across all CSVs.
3. Increment by 1, zero-pad to 3 digits (e.g. `P-011`, `B-017`).
4. This serial becomes the product's permanent ID — it should never be reused, even if the product is discontinued.

## How to update prices / stock

1. Open the relevant CSV (e.g. `data/solar_panels.csv`) in Excel, Google Sheets, or any text editor.
2. Edit the price, stock quantity, or add new rows following the schema above.
3. Commit and push to GitHub.
4. GitHub Pages rebuilds automatically — the live site reflects changes within a minute or two (refresh the page).

## Future: scaling to a central inventory database

When ready to move beyond CSVs, the recommended path is:

1. Keep the same column structure as the foundation for a SQL table (`products` table with these columns + a few inventory-specific ones: `purchase_date`, `vendor_serial_number`, `location`, `condition`, `assigned_to`).
2. Each physical unit gets:
   - A **category-coded serial** (`serial_id` above) — identifies the *product type*.
   - A **unique asset/unit ID** — identifies the *individual physical item* (e.g. `P-001-00734` where `00734` is a per-unit counter), which is what gets paired with the manufacturer's printed serial number.
3. A query for "Panel 50kW" would filter `WHERE category = 'Solar Panel' AND ...` and the UI would simply not render columns/fields that are `NULL` for that category — exactly the "N/A for irrelevant columns" behavior described, achieved naturally by a `LEFT JOIN` or a single wide table with nullable category-specific columns.
4. The current `specifications` field (pipe-separated free text) is a good staging format — when migrating to SQL it can be split into proper typed columns (`voltage`, `current_rating`, `dimensions`, etc.) per category as needed.

## Hosting on GitHub Pages

1. Push this folder to a GitHub repository.
2. Repo Settings → Pages → set source to the branch/folder containing `index.html`.
3. Share the generated `https://<username>.github.io/<repo>/` link with the team.