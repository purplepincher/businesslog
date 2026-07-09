# businesslog (businesslog.ai)

> This repo is **only the landing page** for `businesslog.ai`. It is a thin
> Cloudflare Worker that serves one static HTML file. The product it markets —
> the `businesslog-agent` Python package — lives in a **separate** project and
> is **not** in this repo. If you came here looking for the library, see
> [businesslog-agent on PyPI](https://pypi.org/project/businesslog-agent/).

## What this repo actually is

`businesslog` is the marketing/explainer site for the `businesslog.ai` domain.
It is a [Cloudflare Worker](https://developers.cloudflare.com/workers/)
configured as a static-asset host: the entire runtime logic is five lines that
forward every request to the bound assets binding (`ASSETS`), which serves
`public/index.html`.

```ts
// src/index.ts — the whole Worker
export default {
  async fetch(request: Request, env: { ASSETS: Fetcher }): Promise<Response> {
    return env.ASSETS.fetch(request);
  },
};
```

That single page explains what `businesslog-agent` is, what problem it solves,
and how to install it. It is built on the shared PurplePincher "domain family"
design system (see [`family/`](./family/README.md)).

### The product this page markets

The landing page advertises a Python package called **`businesslog-agent`**,
described as a thin bridge that writes business events (sales, signups, failed
checkouts) as tiles into **PLATO** — the org's tile-based knowledge store — so
the history can be queried across time instead of rotting in a log stream.

| claim on the page | status |
|---|---|
| "`pip install businesslog-agent`" works | ✅ **real today** — the package exists on PyPI as `businesslog-agent` 0.2.0 (Python ≥3.10; depends on `fleet-agent>=0.2.0` and `requests`), owned by `superinstance`. Verified against `pypi.org/pypi/businesslog-agent/json`. |
| The quickstart is "copied verbatim from its README" | ⚠️ **real but unverified from PyPI metadata** — the page's quickstart uses `from businesslog_agent import write_tile, query_metrics` with `query_metrics(room="businesslog", limit=50)`, but the **published PyPI long-description is empty**, so "verbatim from its README" could not be confirmed from PyPI alone. The code is plausibly accurate (the summary mentions "PLATO fleet enabled" and the `fleet-agent` dependency is consistent with a tile-store bridge), but treat the exact signatures as needing a check against the package source, not as confirmed by this repo. |
| It is "a small PLATO domain agent, not a finished analytics product" | ✅ **real / honestly stated** — the page says exactly this in its status banner and honesty note. |
| A larger pipeline (`businesslog-ai` for analysis, `businesslog-app` for structured logging) exists | 🔮 **aspirational** — the page itself says these are "referenced but not yet shipped around it." |

The page's own persistent status banner reads **`AGENT BRIDGE`** and is explicit
that this is a thin agent, not a finished product.

## What this repo is NOT

- ❌ **It does not contain the Python library.** No `.py` files, no
  `pyproject.toml`, no package code. The library's source is a separate project
  (its PyPI metadata points the homepage to `github.com/SuperInstance/businesslog-agent`).
- ❌ **No backend, no API, no database, no connection to PLATO.** The Worker
  does not call out to anything; it only serves the static HTML/CSS. All the
  behavior you see is in the browser.
- ❌ **No test suite.** There is nothing to run here except the page itself.
- ❌ **No build step in this repo.** Although [`family/README.md`](./family/README.md)
  describes a build-time CSS-inlining pattern, in *this* repo the shared
  `family/*.css` has already been hand-inlined into `public/index.html`'s single
  `<style>` block. The deployed artifact is the HTML as it sits in the repo.

## Run it locally

You need the Cloudflare `wrangler` CLI (this repo has no `package.json`;
invoke wrangler directly or via `npx`):

```bash
wrangler dev        # or: npx wrangler@latest dev
# → Ready on http://localhost:8788/
```

Verified pattern: this Worker is byte-identical in logic to its sibling repos
(`dmlog`, `makerlog`); under `wrangler dev` (wrangler 4.107.0) the same shape
boots in ~5 s and serves `/` with **HTTP 200** from `public/index.html` (≈18.7
kB), title *businesslog.ai — Business metrics in PLATO*.

## Deploy it

```bash
wrangler deploy     # or: npx wrangler@latest deploy
```

The Worker name is `businesslog` (see [`wrangler.jsonc`](./wrangler.jsonc));
`compatibility_date` is `2026-07-07`. Deployment (and binding the Worker to the
real `businesslog.ai` route/custom domain) requires your own Cloudflare account
and auth — that part is ⚠️ **conditional on a Cloudflare account**, not
something the repo does for you.

## Repo layout

```
businesslog/
├── public/
│   └── index.html        # the entire landing page (HTML + inlined CSS)
├── src/
│   └── index.ts          # 5-line Worker: env.ASSETS.fetch(request)
├── family/               # shared PurplePincher design system (see its README)
│   ├── README.md         # operator's manual for the family skeleton
│   ├── tokens.css        # shared :root palette + per-site accent variables
│   ├── base.css          # shared reset + component classes (.eyebrow, .chain, …)
│   ├── provenance-panel.css   # the "what's real here" honesty component
│   └── provenance-panel.html  # drop-in honesty-panel partial
├── wrangler.jsonc        # Cloudflare Worker config (name, main, assets binding)
└── .gitignore
```

## How this site fits the family

This is one of several sibling landing-page Workers in the PurplePincher domain
family (e.g. `dmlog`, `makerlog`). Each is its own Worker, its own repo, its own
route, and its own `wrangler.jsonc` — there is deliberately **no** monorepo and
no `deploy-all`.

Each family site swaps exactly **one** accent variable to distinguish itself.
`businesslog` overrides the family accent in its own `:root`:

```css
:root {
  /* businesslog.ai accent override */
  --claw: var(--antifoul);   /* #B0533A — anti-fouling bottom-paint oxide */
}
```

It uses `--antifoul` (a weathered oxide red) — the "working vessel" accent in
the family plan. See [`family/README.md`](./family/README.md) for the full token
and component reference and the rationale for the inline-don't-fetch
architecture.

## Honesty markers at a glance

- ✅ The Worker serves the landing page (the asset-passthrough logic is traced
  in `src/index.ts`; the identical sibling shape was served HTTP 200 under
  `wrangler dev`).
- ✅ The marketed `businesslog-agent` package exists on PyPI (0.2.0).
- ⚠️ The page's "quickstart copied verbatim from its README" could **not** be
  confirmed from PyPI (empty long-description); verify against package source.
- ⚠️ Actually *deploying* to `businesslog.ai` needs a Cloudflare account +
  DNS/route setup not included here.
- 🔮 The `businesslog-ai` (analysis) and `businesslog-app` (structured logging)
  siblings are referenced by the page but explicitly not yet shipped.
