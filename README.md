# PantryMind AI
## Run locally in VS Code
1. Extract the ZIP, then open the `PantryMind-AI` folder in VS Code.
2. Install the free **Live Server** extension by Ritwick Dey.
3. Right-click `index.html` → **Open with Live Server**.
4. Keep an internet connection for the free Tesseract OCR scripts/model, Chart.js, and optional web fonts. No API key, npm installation, paid subscription, backend, or login is required.
5. Use the same browser and the same Live Server address/port each time. Data is stored per browser origin. Do not rely on opening the HTML directly via `file://`.

Alternative: run `python -m http.server 5500` in this folder, then open http://localhost:5500. Use `python3` where necessary.

## Classroom walkthrough (about 3 minutes)
1. Open Settings → Enter demonstration mode. All visible inventory/spending is labelled SAMPLE data. Personal data is kept separately.
2. Overview: show the four metrics, expiry alerts, and six-month spending chart.
3. My pantry: edit Milk's stock and minimum threshold. Explain that quantities are current stock; dates and thresholds are configured by the user.
4. Shopping list: add restock suggestions and check an item off. Checking off does not automatically add inventory.
5. Scan receipt → Use labelled sample receipt → Recognize receipt. This generates a clearly labelled image, then runs actual Tesseract OCR; it does not insert fabricated OCR output.
6. Review text and rows. Correct mistakes, select categories, confirm date and line totals, then save. OCR never fills missing quantities or expiry dates. `Other` category and zero minimum stock are explicit application defaults, not recognized facts.
7. Spending insights: show the purchase ledger and category chart. Delete a pantry entry to demonstrate that past spending remains.
8. Settings: download a JSON backup. Return to personal pantry to show that it is preserved.

## Implementation stages
- Stage 1: responsive shell, overview, manual inventory CRUD/search/filter, purchase ledger, persistent localStorage.
- Stage 2: real Tesseract.js English OCR with progress/errors, camera/file inputs, editable raw text, conservative rule parsing, editable rows and atomic reviewed saves.
- Stage 3: stock-based shopping list, Chart.js analytics, themes, separate demo snapshot, JSON backup/restore, validation and error handling.

## What is AI?
Tesseract uses a trained recognition model to convert image pixels into text. That is the AI component. Parsing prices/quantities, creating minimum-stock suggestions, comparing expiry dates, displaying charts and saving data are ordinary JavaScript automation. There is no conversational AI or paid API.

## Data definitions
- Inventory item = one purchase batch, including current quantity and its own expiry date. A new receipt creates new batches rather than guessing how to merge dates.
- Purchase price = the entire item-line total, NOT the per-unit price. Spending is the sum of recorded line totals, without independently adding taxes, delivery fees or discounts.
- Monthly spending = purchase ledger entries dated in the current local calendar month.
- Expiry alerts = positive-stock entries already expired or expiring within seven days. An entered date is a reminder, not a food-safety assessment.
- Dashboard low stock counts individual entries. Shopping suggestions aggregate same-name/same-unit batches and use the largest configured minimum. Suggested quantity brings stock to minimum + 1 unit.
- Editing an inventory purchase price/date/category updates its linked ledger entry. Changing stock or deleting inventory does not erase history. Removing a ledger entry corrects spending only.
- Prices left blank are excluded from spending, not assumed to be zero.

## OCR limitations and recovery
English printed receipts only. Supported item lines resemble `Milk 2 L 120.00` or `Apples 60.00`. The final number is interpreted as a line total; an explicit preceding count/unit may be extracted. Ambiguous layouts, wrapped names, unit-price columns, tax abbreviations and handwriting require correction. OCR confidence describes text recognition, not parsing correctness. No parser can guarantee arbitrary receipt formats.

Use a bright, straight, tightly cropped photo. JPG/PNG/WebP under 12 MB are supported; convert PDFs or HEIC first. Camera capture is a browser/device capability. On iPhones, the file picker may offer taking a photo even without the capture input. Never save before comparing each row to the receipt.

Internet is needed to fetch the free OCR worker/model and chart library. Inventory works if those libraries fail; chart errors display text totals. No receipt-image upload is made by application code: OCR runs locally in the browser, while libraries/model files are fetched externally. Images and raw OCR text are not persisted or backed up. No offline installation bundle is included.

## Backup and persistence
The main dataset, mode snapshots, and theme are stored in localStorage. Closing/reloading preserves saved data. Clearing browser storage or changing browser/origin does not. Personal and demo modes have separate snapshots; export each mode separately if needed. Restore replaces only the matching active mode after schema validation and confirmation. Backup files contain your grocery and spending information; store them privately.

## Tests
With Node.js 18+ installed, run:

```sh
node --test tests.cjs integration-tests.cjs
```

13 tests passed during development: thresholds, dates, invalid input, receipt parsing/missing values, backup round trip/rejection, demo separation, shopping aggregation/duplicate prevention, persistence, spending preservation and chart calculations. Integration tests use a lightweight DOM stub and do not verify browser rendering or network libraries.

### Local browser acceptance checklist
- Add a grocery, edit quantity, search/filter it, delete it; reload to confirm persistence.
- Check invalid negative quantity, expiry before purchase, and empty name are rejected.
- Scan the sample with real OCR; verify Milk/Rice/etc against the image. Do not expect perfect text.
- Scan your own receipt photo and correct incomplete or incorrect rows. Check a missing quantity blocks saving and a blank price does not fabricate spending.
- Verify all rows save together, with no changes on a validation failure.
- Test chart tooltips, theme switching, hamburger navigation and a 375 px viewport.
- Export a backup, change data, restore it; reject an invalid JSON/schema without losing existing data.
- Test demo → personal → demo isolation.
- Disable network and verify OCR/chart error messages; previously stored inventory should remain usable.

Browser rendering, camera capture, CDN loading and a live Tesseract recognition run could not be verified in the build environment. Run this checklist locally before the school presentation.

## Source files
`index.html` — accessible application shell and inventory modal
`styles.css` — responsive light/dark presentation
`core.js` — shared pure validation/parser/date functions
`app.js` — views, interactions, OCR, charts, persistence
`tests.cjs`, `integration-tests.cjs` — automated checks

Free libraries pinned to Tesseract.js 5.1.1 and Chart.js 4.4.8. Official documentation: https://github.com/naptha/tesseract.js and https://www.chartjs.org/docs/latest/
