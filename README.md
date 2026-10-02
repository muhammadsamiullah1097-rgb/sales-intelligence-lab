# Sales Intelligence Lab 🛒

A fully static, mobile-first web app that explores **real retail transaction data** — with a Customer view and an Owner/Manager view, plain-language insights, and recommendations computed from the data.

Live demo: *(to be deployed on GitHub Pages as its own repo)*

## Data provenance

| Field | Value |
|---|---|
| Dataset | **Online Retail** — real transactions from a real UK-based online retailer |
| Source | UCI Machine Learning Repository |
| Download URL | https://archive.ics.uci.edu/static/public/352/online+retail.zip |
| License | **CC BY 4.0** |
| Period | 1 Dec 2010 – 9 Dec 2011 |
| Full file | 541,909 rows, 8 columns (InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country) |
| Downloaded | 2026-10-02 |
| This site's sample | **40,000 rows** — deterministic random sample via `pandas.sample(n=40000, random_state=42)` from the 406,829 rows with a non-null CustomerID; covers the full period, 3,791 customers, 37 countries |

### Sampling / cleaning decisions (documented so results are reproducible)
1. Downloaded the official zip, extracted `Online Retail.xlsx`.
2. Dropped rows with missing `CustomerID` (135,080 guest-checkout rows — needed for the customer lookup view), missing `InvoiceNo`, or missing `Description`.
3. Rows with **negative Quantity** (cancellations/returns) and **zero UnitPrice** (adjustments) were **kept** — they are real business events and the app analyzes them.
4. Took a deterministic random sample of 40,000 rows with seed 42 (`data/data.js`, ~3.9 MB). Re-running with the same seed reproduces the file exactly.

## How to refresh the data
1. Download the zip from the URL above and unzip it.
2. In this folder's `data/` directory, run a script like:
   ```python
   import pandas as pd, json
   df = pd.read_excel("Online Retail.xlsx", engine="openpyxl")
   df = df.dropna(subset=["CustomerID","InvoiceNo","Description"])
   df["CustomerID"] = df["CustomerID"].astype(int)
   df["InvoiceNo"] = df["InvoiceNo"].astype(str)
   sample = df.sample(n=40000, random_state=42).sort_values("InvoiceDate")
   recs = [[r.InvoiceNo, r.InvoiceDate.strftime("%Y-%m-%dT%H:%M"), int(r.CustomerID),
            str(r.StockCode), str(r.Description), int(r.Quantity),
            round(float(r.UnitPrice),2), str(r.Country)] for r in sample.itertuples()]
   open("data.js","w").write("var SALES_DATA = " + json.dumps(recs, separators=(",",":")) + ";\n")
   ```
3. Keep the column order `[InvoiceNo, Date, CustomerID, StockCode, Description, Quantity, UnitPrice, Country]` — the app expects it.
4. Update the download date and row counts in the "Data source" card in `index.html`.

## Project structure
```
sales-intelligence-lab/
├── index.html   # whole app: customer view, owner dashboard, charts, recommendations
├── data/
│   └── data.js  # var SALES_DATA = [40,000 rows]; works offline (no fetch needed)
└── README.md
```

## Features
- **Customer view**: enter a Customer ID → orders, total spent, favourite product, and a plain-language summary ("you are in the top 10% of customers").
- **Owner view**: revenue KPIs, monthly revenue, top products, revenue by country, cancellation/return analysis, product search, CSV export of filtered transactions.
- **Recommendations panel**: every tip is *computed from the data* (best/slowest month ratio, highest return-rate product, fastest-growing country, busiest weekday, revenue concentration, median order value) — no hardcoded fluff.
- **Honesty**: a visible data-source card with source, license, sample method, and a "📸 data snapshot — not live" badge. No fake LIVE indicators anywhere.
- **Resilience**: Chart.js loads from CDN, but if it fails the app still shows all numbers with a polite message instead of a blank page.

## QA (2026-10-02)
- JSON in `data.js` parses (40,000 rows).
- Inline JS passes `node --check`.
- Served with `python3 -m http.server`; `curl` returns HTTP 200 and all key sections present.
- Mobile-first CSS with `@media (max-width:560px)` rules; big touch targets (min 48–56px).
