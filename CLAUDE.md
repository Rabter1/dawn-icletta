# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This is a Shopify Online Store 2.0 theme — a customized fork of [Shopify Dawn](https://github.com/Shopify/dawn) for the **icletta** store. The active development branch is `icletta-dev`; commits regularly arrive from the Shopify Theme Editor with messages like `Update from Shopify for theme dawn-icletta/icletta`, meaning merchants can edit the theme outside the repo and changes get pushed back. Watch for editor-driven churn in `config/settings_data.json`, `templates/*.json`, and section group JSON files.

There is no Node/npm build step. Liquid + CSS + JS are served as-is by Shopify; assets are referenced via `{{ 'file.ext' | asset_url }}`.

## Common commands

All workflow tooling is provided by the [Shopify CLI](https://shopify.dev/docs/themes/tools/cli).

```bash
shopify theme dev          # local dev server with hot reload against a dev store
shopify theme check        # lint Liquid (Theme Check) — config in .theme-check.yml
shopify theme push         # push to a theme on the store (use --unpublished for safety)
shopify theme pull         # pull theme-editor edits down before working locally
```

CI (`.github/workflows/ci.yml`) runs **Theme Check** and **Lighthouse CI** on every push — there are no unit tests.

Prettier (`.prettierrc.json`) is configured with `printWidth: 120`, `singleQuote: true`, but `singleQuote: false` for `*.liquid` (Liquid uses double quotes).

## Architecture

Standard Shopify theme directory layout — Shopify's runtime resolves files by directory, so do not rename or move them:

- **`layout/`** — top-level page wrappers. `theme.liquid` is the global shell (loads fonts, CSS variables from color schemes, core JS); `password.liquid` is the password-page shell.
- **`templates/`** — JSON templates that compose sections per page type (e.g. `product.json`, `collection.json`). Custom alternate templates use suffixes Shopify exposes in admin: `product.kette.json`, `page.themenwelt-*.json`, `collection.filtertest.json`, etc.
- **`sections/`** — Liquid sections referenced by JSON templates. `*-group.json` (e.g. `header-group.json`, `footer-group.json`) are section groups inserted into `theme.liquid`.
- **`snippets/`** — reusable Liquid partials rendered with `{% render 'name' %}`. Includes icons (`icon-*.liquid`), product UI pieces, and custom integrations like `bc_banner.liquid` (BörgerCookie/GDPR), `doofinder-script-tag.liquid` (Doofinder search), and `country-localization.liquid`.
- **`assets/`** — flat directory of all CSS, JS, images, fonts. No bundler — every file is independently served. Naming conventions: `component-*.css` (reusable UI), `section-*.css` (section-specific), `template-*.css` (template-specific). Site-specific overrides live in `custom-icletta.css`.
- **`config/`** — `settings_schema.json` defines the theme settings UI; `settings_data.json` stores the merchant's saved values (frequently overwritten by the editor).
- **`locales/`** — buyer-facing strings (`{lang}.json`) and merchant/editor strings (`{lang}.schema.json`). `en.default.json` and `en.default.schema.json` are the source of truth; `translation.yml` configures Shopify's translation pipeline.

### How a page renders

1. Shopify picks a `templates/<type>.json` (or `<type>.<suffix>.json` if requested).
2. `layout/theme.liquid` wraps it, including the `header-group` and `footer-group` section groups.
3. The JSON template lists `sections` (with settings + block order); each section is a `sections/*.liquid` file.
4. Sections render `snippets/*.liquid` partials and reference `assets/` via `asset_url`.

### JS conventions (vanilla, no framework)

Per Dawn's "Web-native, no abstractions" principle, JS is plain ES modules using **Custom Elements** (e.g. `cart-drawer.js`, `product-form.js`, `quantity-popover.js`). Cross-component messaging goes through `assets/pubsub.js`. Shared constants live in `assets/constants.js`. Files are loaded with `defer` from `theme.liquid` or directly from sections — there is no bundler, so each `<script>` tag costs a request.

### Custom integrations specific to this fork

- **Doofinder** site search — `snippets/doofinder-script-tag.liquid`.
- **BörgerCookie / GDPR cookie banner** — `snippets/bc_banner.liquid` (reads `shop.metafields.bc_cookie`).
- **Custom themed landing pages** under `templates/page.themenwelt-*.json` and `page.t-welt-*.json`.
- **`load-metafields.liquid` / `zload-metafields.liquid`** — central metafield preloading; check both when adding metafield-driven features.
- **`custom-icletta.css`** — store-specific CSS overrides; prefer adding overrides here rather than editing core component CSS so future Dawn upstream merges stay clean.

## Working with this codebase

- **Don't fight the theme editor.** `config/settings_data.json` and the JSON templates in `templates/` and section group files are routinely rewritten by merchants in the Shopify admin and pushed back via the auto-sync. Avoid hand-editing them unless the change is structural; settings changes belong in the editor.
- **Pull before edit.** Run `shopify theme pull` against the live theme before starting work to avoid clobbering editor changes.
- **Stay Web-native.** No npm dependencies, no frameworks, no polyfills. Add a Custom Element + a small CSS file rather than reaching for a library.
- **Keep CSS scoped.** New section CSS goes in `assets/section-<name>.css` and is loaded by that section only; component CSS is shared.
- **Upstream Dawn.** The repo is a fork — when touching core files, consider whether the change should live in `custom-icletta.css` or a new snippet to ease future `git pull upstream main` from `Shopify/dawn`.
