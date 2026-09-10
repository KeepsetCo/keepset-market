# keepset-market

The menu a customer sees when they scan the one QR code at the booth.

It replaces five per-product QR signs with a single permanent one. The QR points here rather
than at a product, so the offering can change between markets without reprinting anything —
that is the actual prize, more than the tidiness of one code.

Deployed to Cloudflare Pages at **make.keepsetco.com**.

## This repo is public, and must stay safe to be

Booth tokens are **never** in here. They are issued at request time by the booth app's scan
redirect (`booth-app/app/routes/booth.$handle.scan.tsx`), which is the invariant the whole
in-person design rests on: photographing the QR at a market gets you a URL that does nothing
on its own.

What ships here is five URLs and some markup. If anything in this repo ever needs a
credential, it does not belong in this repo.

## Structure

There is no build step. `public/` is the site.

```
public/
  index.html    the menu
  keepset.css   brand tokens, copied from keepset-design/tokens
  logo.svg      keepset-design/brand/assets/logo-lockup-stacked.svg
  fonts/        Cormorant Garamond and Jost, the same two files the storefront serves
  _headers      cache rules — fonts forever, HTML never
```

`keepset.css` copies its colour and type values out of `keepset-design/tokens/build/tokens.css`
rather than importing them, because this repo deliberately has no dependency on the design
repo and no build. That file is generated and remains the source of truth: if a value here
disagrees with it, it is this file that is wrong.

## The five links

Every entry links to the booth app's scan route, never to a storefront URL:

```
https://booth.keepsetco.com/booth/<handle>/scan?shop=4gnmra-cv.myshopify.com
```

The handles are hardcoded. They change rarely, and a wrong one redirects to the shop front
rather than erroring. To find one, open the link in the booth app at
`booth.keepsetco.com/app` and read the handle out of the address bar — `/app/booth/<handle>`.

Switching a link **inactive** does not break this page. The scan still redirects; it simply
stops attaching a token, so the customer reaches the same product page and checks out by card
like anyone shopping from home. `active` is a pack-down switch, not an in-market one.

## What is deliberately not here

**Prices.** They live in Shopify and on the printed sign. A third copy is the one that goes
stale silently and is wrong in front of a customer. A scan lands on the product page, which
shows the authoritative price one tap later — so a price change never needs a deploy here.

## The printed sign

Lives in the design repo, at `keepset-design/physical/booth-sign/`. It carries the prices, the
QR, and the URL in type underneath it. If the hostname above ever changes, the QR in that
folder has to be regenerated and the signs reprinted.

## Deploying

Cloudflare Pages, building from `main`.

| Setting | Value |
|---|---|
| Framework preset | None |
| Build command | *(leave empty)* |
| Build output directory | `public` |
| Root directory | `/` |

There is no build step, so a deploy is an upload. Then add `make.keepsetco.com` as a custom
domain on the project and put a CNAME at Namecheap — where the zone lives — pointing `make`
at the project's `pages.dev` hostname.

### Why this costs nothing, checked rather than assumed

Verified against Cloudflare's own docs on 2026-09-10:

- **Static assets are free and unlimited.** The Workers pricing page says exactly that —
  *"Requests to static assets are free and unlimited"* — and *"There are no additional charges
  for data transfer (egress) or throughput (bandwidth)."* This site is nothing but static
  assets.
- **500 builds a month** on the free plan, one at a time. A deploy per push to `main`; this
  repo will not see 500 pushes in a month.
- **20,000 files** per deployment and **25 MiB** per file. This repo has eleven files and the
  largest is 84 KB.
- **100 custom domains** per project, **100 projects** per account.
- **The certificate is issued and renewed by Cloudflare at no cost.**

Two constraints worth knowing before anyone changes the plan:

- **A subdomain can live on external DNS; an apex cannot.** Pages will happily serve
  `make.keepsetco.com` with the zone still at Namecheap. Serving `keepsetco.com` itself from
  Pages would require moving the whole zone to Cloudflare — and that apex belongs to Shopify.
  This is the reason the menu is a subdomain, beyond it simply reading better on a sign.
- **A CAA record that excludes Cloudflare would block the certificate silently.**
  `keepsetco.com` has no CAA records at all as of 2026-09-10, so nothing is in the way. If one
  is ever added, Cloudflare has to be an allowed issuer or this site loses HTTPS.
