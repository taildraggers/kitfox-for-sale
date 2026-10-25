# Kitfox For Sale

Daily aggregator of Kitfox aircraft classified listings from
[Barnstormers.com](https://www.barnstormers.com), published as a static page
(`docs/index.html`) meant to be embedded via `<iframe>` on taildraggers.com.

## How it works

- `scraper/barnstormers.py` runs a Barnstormers advanced search for ads whose
  headline contains **"Kitfox"**, and walks every page of results: Barnstormers
  shows 25 ads per page with a `&page=N` pager, so the scraper requests page 1, 2, 3,
  ... until a page turns up no new ads. It then visits each listing's detail page to
  pull out the price, location, and posted date (falling back to regex heuristics over
  the visible text since the site doesn't expose structured data). The title is
  derived from the listing URL's own SEO slug, since every detail page shares one
  generic `<title>`/`<h1>`.
- Only whole-aircraft-for-sale listings are published. Each ad's title must name a
  Kitfox; titles that read as parts, accessories, services, raffles, or an unbuilt
  kit are dropped (phrases like "on floats" or "kit built" are allowed - see
  `ALLOWED_PHRASES` in `scraper/common.py`). Every surviving listing's title is
  rewritten to a canonical **`YEAR Kitfox MODEL`** form, where MODEL is the
  generation (`Model 1`-`Model 4`, `Series 5`-`Series 7`) and/or named variant
  (Super Sport, STi, Speedster, Classic IV, Outback, Safari, Vixen, Voyager, Lite,
  XL) the ad states - e.g. `2015 Kitfox Series 7 Super Sport` - dropping whatever
  the ad doesn't state; see `_SERIES_RES`/`_VARIANT_RULES` in
  `scraper/barnstormers.py`.
- Ads priced under $1,000 are dropped as parts listings (`MIN_AIRCRAFT_PRICE` in
  `scraper/common.py`).
- `main.py` runs the scraper, de-duplicates results (including the same ad reposted
  under a new ad number: identical title, price, and location), and renders them into
  `docs/index.html` titled **"Other Kitfox Ads on the Web"**, with one row per
  listing: Title (linked to the original ad), Price, Location, Date Posted, and Site
  Posted On. Links use `rel="noopener noreferrer"` and the page sets a
  `no-referrer` meta policy, so Barnstormers never sees that the click came from
  taildraggers.com.
- `.github/workflows/daily-scrape.yml` runs the whole thing once a day (13:00 UTC),
  commits the regenerated `docs/index.html` if it changed, and can also be triggered
  manually from the Actions tab (`workflow_dispatch`).

## One-time setup: enable GitHub Pages

This repo publishes `docs/index.html` as a plain static file — GitHub Pages just needs
to be pointed at it once:

1. Go to **Settings → Pages** in this repository.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Branch: `main`, folder: `/docs`. Save.
4. GitHub will publish the page at `https://taildraggers.github.io/kitfox-for-sale/`
   (may take a minute or two the first time).

## Embedding on taildraggers.com

```html
<iframe
  src="https://taildraggers.github.io/kitfox-for-sale/"
  title="Other Kitfox Ads on the Web"
  style="width: 100%; height: 800px; border: 0;"
  loading="lazy">
</iframe>
```

## Running locally

```bash
pip install -r requirements.txt
python main.py
```

This writes/overwrites `docs/index.html`.

## Notes

- If Barnstormers changes its markup or is briefly unreachable, the run logs will
  show a `[warn]`/`[error]` line pointing at what broke rather than failing silently.
- The scraper identifies itself with a browser-like `User-Agent` and adds a short
  delay between requests to be polite to the site.
