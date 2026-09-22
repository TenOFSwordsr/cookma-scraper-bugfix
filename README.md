# Cookma Price Scraper Bugfix with Standalone Tests

The corrected revision of the two Persian product-price scrapers that feed the WooCommerce store at
`shifteryadak.ir`, plus a network-free test file and a runner that does not need pytest installed.
This is the newest of the three copies of these scripts on the machine; the price-parsing bugs the
tests describe are the reason the revision exists.

**Suggested repo name:** `cookma-scraper-bugfix`
**Stack:** Python 3.11 (per the `__pycache__` tags), Playwright async API, requests, BeautifulSoup4; no manifest file
**Status:** finished
**Last modified:** 2026-09-02

## What it does

- `Final Files/price.py` - crawls `https://cookmamotor.com/home/productList` by driving the site's own
  `reloadDataProduct(page, size)` AJAX call through Playwright (10 rows per page, `MAX_PAGES = 500`
  safety cap), parses the returned table, converts rial to toman, and writes
  `~/Desktop/cookma/cookma_YYYY-MM-DD.csv` with headers
  `عنوان محصول, دسته‌بندی, ویژگی‌ها, قیمت (تومان), شناسه (Hash ID), لینک محصول`.
  Functions: `parse_price_digits`, `rial_to_toman`, `generate_hash_id`, `build_features`,
  `extract_link`, `parse_table`, `reload_and_wait`, `fetch_all_products`, `save_to_csv`.
  Falls back to a system Chrome/Edge binary when one is found, else Playwright's own Chromium.
- `Final Files/other_sites_scraper.py` - the four competitor shops: static WooCommerce pagination for
  `mrkasket.com` and `starcyclet.com` (`MAX_STATIC_PAGES = 200`, `STATIC_RETRIES = 3`,
  `extract_woo_price` preferring the `<ins>` sale price), and infinite-scroll browser crawls for
  `kolahkasket.com` and `gazzero.com` capped at `MAX_PRODUCTS_PER_DYNAMIC_SITE = 2500`. Output:
  `~/Desktop/other_sites/other_sites_YYYY-MM-DD.csv` with
  `سایت, عنوان محصول, قیمت (تومان), لینک محصول`.
- `test_scrapers.py` - 19 checks on the pure functions only, loading both modules by path. The named
  regressions are `test_rial_to_toman_ignores_parenthesized_suffix` (the old regex glued
  `250,000 (2)` into `2500002`) and `test_clean_price_first_number_only`; plus hash-id stability across
  category/brand, `parse_table` tolerance of a missing `<tbody>`, link resolution, and the
  unavailable-price sentinels `ناموجود` / `غیرفعال` / `تماس بگیرید`.
- `run_tests.py` - discovers every `test_*` callable in `test_scrapers` and reports
  `PASS / FAIL / ERROR` with a totals line and exit code.

## Layout

```
Final Files/price.py                 cookma scraper (fixed revision)
Final Files/other_sites_scraper.py   four-shop scraper (fixed revision)
test_scrapers.py                     pure-function tests, no network
run_tests.py                         pytest-free runner
```

## Running it

```bash
python run_tests.py                                   # tests only, no browser
pip install playwright requests beautifulsoup4 && python -m playwright install chromium
export COOKMA_PHPSESSID=<value>                       # instead of editing the source
python "Final Files/price.py"
python "Final Files/other_sites_scraper.py"
```

## Notes

- `Final Files/price.py:14` keeps a live cookma `PHPSESSID` value as the fallback literal (and warns at
  startup when the env var is unset). Remove it before publishing.
- Output paths are not configurable: both scripts write under the current user's Desktop
  (`~/Desktop/cookma`, `~/Desktop/other_sites`) and log to `scraper_log.txt` beside the cookma output.
- Duplication: `Documents\Projects\client-fix\final-files1\Final Files` and
  `Documents\Projects\shiftery-fix\final-files\Final Files` hold the older May 17 revision of the same
  pair (`price.py` 7.5 KB vs 15 KB here), which still has the glued-number parsing bug and covers only
  two of the four shops. A PHP port of this logic lives in
  `Documents\Projects\shiftery-fix\grimoire-scrapers.php`.
- No `requirements.txt`; the dependency list above is read from the imports, and `requests` plus
  `bs4` are only needed by the other-sites script.
