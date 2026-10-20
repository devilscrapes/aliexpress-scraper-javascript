<p align="center"><a href="https://apify.com/DevilScrapes/aliexpress-products-scraper"><img src="assets/banner.svg" alt="AliExpress Products Scraper — JavaScript example" width="100%"></a></p>

# AliExpress Scraper for JavaScript and Node.js: product data for apps and price alerts

<p align="center">
<a href="https://apify.com/DevilScrapes/aliexpress-products-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/aliexpress-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/aliexpress-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

This ES module lets a Node.js app use the **[AliExpress Products Scraper](https://apify.com/DevilScrapes/aliexpress-products-scraper)** through the official `apify-client` package. Hand it keywords, item URLs or product IDs; Apify runs the Actor and you receive AliExpress products as plain objects with a numeric price, the currency code, star rating, parsed sold count, canonical item URL and image links. Your server never requests AliExpress pages itself, so the same code runs unchanged in a container, a background worker or a serverless handler. The CLI saves `results.json` and `results.csv`, and inside your service you can use the returned array directly.

> **Actor:** [AliExpress Products Scraper](https://apify.com/DevilScrapes/aliexpress-products-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [aliexpress-scraper-python](https://github.com/devilscrapes/aliexpress-scraper-python)

## 🔥 What you get

- **Three ways in**: search keywords, `aliexpress.com/item/...` links and bare numeric product IDs can be mixed in a single run.
- **Each product appears once**: a `product_id` already written in the run is skipped, even when it turns up again through another keyword, a URL or the ID list.
- **Demand signal on search rows**: the "sold" badge from the results page is parsed into an integer `order_count`, alongside price, currency, star rating and images.
- **Plausibility checks on every row**: the price must be above zero and `product_url` must contain the row's own numeric ID, so a page that loaded without real product data never becomes a record.
- **Country-matched routing**: by default the Actor uses residential Apify Proxy in `shippingDestinationCountry` and, for the countries it has a currency for, skips a keyword whose first results page is priced in the wrong one.
- **Clear failure semantics**: a keyword with no matches finishes with zero rows and a status message; the run fails only when every one of its inputs went unresolved.

## 🎯 Use cases

### Run a price-drop alert service

Store the last price per `product_id`, re-run the same IDs on a schedule and notify users when an item gets cheaper. The JavaScript recipe below contains the comparison step; plugging in email or a webhook is the remaining work.

### Import products into a storefront

Dropshipping and catalog tools can send a pasted AliExpress link to a backend route, run the Actor with that URL and prefill a product form with title, price and images. The canonical `product_url` makes a stable foreign key.

### Expose product search through your own API

Put the core call behind an endpoint in Express, Fastify or a cloud function. Clients send a keyword, your code returns the products, and rate limits, caching and auth stay under your control.

### Keep a margin dashboard current

Join each refreshed AliExpress price with your own selling price to show margin per item on an internal dashboard. Items whose cost rises past a threshold can be flagged automatically after each scheduled run.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/aliexpress-scraper-javascript.git
cd aliexpress-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [AliExpress Products Scraper](https://apify.com/DevilScrapes/aliexpress-products-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

In application code the whole integration is the block below. `call()` returns once the Actor run has finished, and `listItems()` reads its default dataset.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/aliexpress-products-scraper').call({"searchKeywords": ["wireless earbuds"], "productUrls": [], "productIds": [], "maxResultsPerKeyword": 10, "shippingDestinationCountry": "US", "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` submits `input.json` to the Actor unchanged. Out of the box it looks up one keyword, keeps ten products and prices them for the US. Fill `productIds` with the items you track to refresh known products, and remember that every ID or URL costs one browser render, so lookups by ID are slower than a keyword search.

```json
{
  "searchKeywords": [
    "wireless earbuds"
  ],
  "productUrls": [],
  "productIds": [],
  "maxResultsPerKeyword": 10,
  "shippingDestinationCountry": "US",
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `searchKeywords` | array | no | `[]` | — |
| `productUrls` | array | no | `[]` | — |
| `productIds` | array | no | `[]` | — |
| `maxResultsPerKeyword` | integer | no | `20` | — |
| `shippingDestinationCountry` | string | no | `"US"` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

Results come back as one object per product. `price` is already a number, but currencies are not converted, so read it together with `currency`. Expect `null` in `seller_name`, `category` and `shipping_cost_estimate`, and an empty `variants` array, until a later Actor version fills them.

| Field | Type | Always present |
|---|---|---|
| `product_id` | string | yes |
| `title` | string | yes |
| `price` | number | yes |
| `currency` | string | yes |
| `rating` | number or null | no |
| `review_count` | integer or null | no |
| `order_count` | integer or null | no |
| `variants` | array | yes |
| `shipping_cost_estimate` | number or null | no |
| `shipping_destination` | string | yes |
| `product_url` | string | yes |
| `image_urls` | array | yes |
| `seller_name` | string or null | no |
| `category` | string or null | no |
| `search_keyword` | string or null | no |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.05 |
| Product scraped | $0.002 |

Prices as configured on 2026-09-01. The [Actor page](https://apify.com/DevilScrapes/aliexpress-products-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

This Node.js script, using built-in modules only, compares the latest prices with `last-prices.json`, lists new products and those at least 5% cheaper than last time, then saves the new snapshot for the next run.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const products = JSON.parse(await readFile('results.json', 'utf8'));
const previous = JSON.parse(await readFile('last-prices.json', 'utf8').catch(() => '{}'));

const changes = [];
for (const product of products) {
    const before = previous[product.product_id];
    if (before === undefined) {
        changes.push({ kind: 'new', id: product.product_id, price: product.price, url: product.product_url });
    } else if (product.price <= before * 0.95) {
        const dropPct = Math.round((1 - product.price / before) * 100);
        changes.push({ kind: `-${dropPct}%`, id: product.product_id, price: product.price, url: product.product_url });
    }
}

console.table(changes);
const snapshot = { ...previous };
for (const product of products) snapshot[product.product_id] = product.price;
await writeFile('last-prices.json', JSON.stringify(snapshot, null, 2));
```

## ❓ FAQ

### How do I avoid paying twice for the same product?

Within one run, duplicates are skipped automatically, across keywords and input lists alike. Across runs that is your decision: query only the IDs you need, or store `product_id` values and filter before you call the Actor.

### Why might a product ID or URL produce no row?

Direct lookups render the item page, and a render can fail. The Actor retries once with a new session; if the page still yields nothing, that item is skipped, you are not charged for it, and the run log names its ID.

### Can I call it from TypeScript?

Yes. `apify-client` comes with type definitions. Paste the core call into a `.ts` module and describe the product row with an interface built from the output table.

### What does the example run cost?

One run start plus one charge per product written, at the prices in the table below; ten products come to about $0.07. Pass `--max-charge` to keep tests within a budget.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/aliexpress-products-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [AliExpress Products Scraper on Apify Store](https://apify.com/DevilScrapes/aliexpress-products-scraper)
- [Python version of this example](https://github.com/devilscrapes/aliexpress-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [eBay Product & Listings Scraper](https://apify.com/DevilScrapes/ebay-product-listings-scraper) — the same products' asking prices on eBay.
- [Mercado Libre Search Results Scraper](https://apify.com/DevilScrapes/mercadolibre-scraper) — marketplace search results for Latin American buyers.
- [Shopify Store Leads Scraper](https://apify.com/DevilScrapes/shopify-store-leads-scraper) — find the Shopify stores that resell such items.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/aliexpress-products-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with AliExpress or Alibaba Group.
