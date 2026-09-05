---
name: campbell-soup-find-recipes
description: Search and retrieve recipes from The Campbell's Company recipe catalog over its public WordPress REST API — no key required.
api: campbell-soup:campbells-content-api
operations:
  - getRecipe
  - getRecipeId
  - getCscRecipeCollection
  - getMediaId
generated: '2026-09-05'
method: generated
source: openapi/campbell-soup-content-api-openapi.yml
---

# Find Campbell's recipes

The Campbell's recipe catalog is readable anonymously at `https://www.campbells.com/wp-json/wp/v2`.
There is no key, no signup and no quota. 316 recipes were live on 2026-09-05.

## 1. Search the catalog — `getRecipe`

```
GET https://www.campbells.com/wp-json/wp/v2/recipe?search=chicken&per_page=20
```

Useful parameters (all declared on the route): `search`, `per_page` (max 100), `page`, `slug`,
`order`, `orderby`, `after` / `modified_after`, `include`, `exclude`, and
`csc_recipe_collection` to scope to a curated collection.

Read pagination off the response headers, not the body:

- `X-WP-Total` — total matches
- `X-WP-TotalPages` — pages at the current `per_page`
- `Link` — RFC 8288 `rel="next"` / `rel="prev"`

Stop when `Link` no longer carries a `next`. Do not compute page counts yourself.

## 2. Narrow by collection — `getCscRecipeCollection`

```
GET https://www.campbells.com/wp-json/wp/v2/csc_recipe_collection?per_page=100
```

Take the numeric `id` of a term ("One Pot Recipes", "Casseroles & Bakes") and pass it back:

```
GET https://www.campbells.com/wp-json/wp/v2/recipe?csc_recipe_collection=<term_id>
```

`csc_recipe_collection` is the **only** taxonomy registered on recipes. There is no ingredient,
cuisine or dietary axis on the recipe type.

## 3. Fetch one recipe — `getRecipeId`

```
GET https://www.campbells.com/wp-json/wp/v2/recipe/61284
```

Add `?_embed` to inline the featured image and author instead of following `_links` yourself.
Add `?_fields=id,slug,title,link,modified` to keep responses small — the default payload includes
rendered HTML.

## Rules

- **Content is rendered HTML.** `title.rendered`, `content.rendered` and `excerpt.rendered` are
  HTML strings with entity escapes (`&#038;`), not plain text. Unescape before use.
- **No nutrition and no UPC.** The retired Campbell's Kitchen API returned Nutrition Facts panels
  and supported UPC lookup. Neither exists on this surface. Do not tell a user you can look up
  nutrition data here.
- **You cannot join recipes to products.** No relation is declared between the `recipe` and
  `product` types. "Which recipes use this Campbell's product?" is not answerable from this API.
- **Honour the cache.** Responses carry `cache-control: max-age=600`. Campbell's publishes no rate
  limit and returns no rate-limit headers, so use the 10-minute window and the `crawl-delay: 10`
  in `robots.txt` as your pace.
- **Errors are not RFC 9457.** A failure returns `{"code":...,"message":...,"data":{"status":...}}`.
  Branch on `code`, not on message text. See `errors/campbell-soup-problem-types.yml`.
- **Read-only.** Write methods exist in the contract but the live surface answers `Allow: GET` to
  anonymous callers, and Campbell's issues no credentials. Never attempt a write.
