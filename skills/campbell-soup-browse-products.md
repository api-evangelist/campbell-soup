---
name: campbell-soup-browse-products
description: Browse and filter The Campbell's Company product catalog across its brand, category, tag and dietary taxonomies over the public WordPress REST API.
api: campbell-soup:campbells-content-api
operations:
  - getProduct
  - getProductId
  - getCscProductLine
  - getCscProductCat
  - getCscProductTag
  - getCscDietaryAlternative
  - getSearch
generated: '2026-09-05'
method: generated
source: openapi/campbell-soup-content-api-openapi.yml
---

# Browse Campbell's products

210 products were live on `https://www.campbells.com/wp-json/wp/v2/product` on 2026-09-05.
Anonymous, no key.

## 1. Resolve a taxonomy term first

Products are the most heavily classified type in the graph — five taxonomies are registered on
them. Filtering takes **numeric term ids**, not names, so resolve first:

```
GET https://www.campbells.com/wp-json/wp/v2/csc_product_line?per_page=100      # brand / product line
GET https://www.campbells.com/wp-json/wp/v2/csc_product_cat?per_page=100       # category
GET https://www.campbells.com/wp-json/wp/v2/csc_product_tag?per_page=100
GET https://www.campbells.com/wp-json/wp/v2/csc_dietary_alternative?per_page=100
```

> `csc_main_ingredient` is registered and exposed but returned **zero terms** on 2026-09-05.
> Ingredient filtering is declared in the contract and inert in practice. Do not build on it
> without re-checking that it is populated.

## 2. Filter the catalog — `getProduct`

```
GET https://www.campbells.com/wp-json/wp/v2/product?csc_product_line=<id>&per_page=50
```

Filters compose. Use `tax_relation=AND|OR` to control how multiple taxonomy filters combine, and
`<taxonomy>_exclude` to subtract terms. Keyword search is `?search=`.

## 3. Fetch one product — `getProductId`

```
GET https://www.campbells.com/wp-json/wp/v2/product/63097?_embed
```

## 4. When a product is missing — `getSearch`

Some products live under the separate `external_product` type and will not appear in
`/wp/v2/product`. Cross-type search finds both:

```
GET https://www.campbells.com/wp-json/wp/v2/search?search=onion+soup+mix
```

Each hit carries `subtype` (`product` vs `external_product`) — read it and follow the
`_links.self` href rather than assuming the collection.

## Rules

- **No GTIN/UPC field.** Barcode lookup is not possible on this surface, despite the retired
  Campbell's Kitchen API having supported it.
- **No pricing, no stock, no store locator** in the contract. This is marketing content.
- **Sibling brands are separate hosts, same contract.** `https://www.snydersofhanover.com/wp-json`
  (37 products) and `https://www.pacefoods.com/wp-json` (28 products) run the identical `wp/v2`
  shape with their own data. Swap the host, keep the paths. Verify before relying on it —
  `www.prego.com`, `www.v8juice.com` and `www.swansonbroth.com` return an HTML shell rather than
  JSON, and `www.thecampbellscompany.com` serves the route document but 403s every collection.
- **Read-only**, same as recipes. Never attempt a write.
- Honour `cache-control: max-age=600`; there is no published rate limit and no rate-limit header.
