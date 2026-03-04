# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**The Fall** is a custom Shopify theme for a bridal/designer fashion store, forked from Shopify's Dawn v15.4.0. It follows a web-native, HTML-first architecture using Liquid templates with vanilla JavaScript (no frameworks).

- **Dev store**: `the-fall-dev.myshopify.com`
- **Origin**: `git@github.com:ahref13/dawn.git`
- **Upstream**: `https://github.com/Shopify/dawn.git` (synced periodically)

## Commands

| Command | Purpose |
|---------|---------|
| `npm run build:css` | Compile Tailwind CSS in watch mode (`assets/app.css` → `assets/output.css`) |
| `shopify theme dev --store the-fall-dev.myshopify.com` | Start local dev server |
| `shopify theme check` | Lint theme (Liquid, performance, best practices) |
| `shopify theme push --development` | Push to development theme on store |

## Architecture

### Rendering Model
All HTML is server-rendered by Shopify via Liquid. JavaScript is used sparingly for progressive enhancement only (no render-blocking JS, no DOM manipulation before user input).

### Directory Layout
- **`layout/`** — Root layouts (`theme.liquid` is the main entry point)
- **`sections/`** — Modular page sections (Online Store 2.0 drag-and-drop). Custom sections use `tf-` prefix
- **`snippets/`** — Reusable Liquid partials included via `{% render %}`
- **`templates/`** — JSON templates that compose sections into pages. Includes `customers/` and `metaobject/` subdirectories
- **`assets/`** — CSS and JS files. `app.css` is the Tailwind input; `output.css` is generated (do not edit directly)
- **`config/`** — `settings_schema.json` (theme customizer definition) and `settings_data.json` (current values)
- **`locales/`** — 53+ translation files; `en.default.json` is the source of truth

### Styling
- **Tailwind CSS v4** compiled via `@tailwindcss/cli`. Config scans `layout/`, `sections/`, `snippets/`, and `templates/`
- **CSS custom properties** for color schemes, defined in `settings_schema.json` and applied in `theme.liquid`
- **Component CSS** files (`tf-*.css`) for complex custom sections

### JavaScript
Vanilla JS only — no frameworks. Key files:
- `global.js` — Core utilities (`HTMLUpdateUtility`, `SectionId`, focusable element helpers)
- `pubsub.js` — Pub/sub event system for cross-component communication
- `product-form.js` / `product-info.js` — Product page logic
- `cart-drawer.js` — AJAX cart drawer
- `facets.js` — Collection filtering
- `animations.js` — Scroll-triggered animations (toggled via theme settings)

### Custom Sections (`tf-*`)
The Fall adds 24 custom sections for: hero animations, designer carousels/galleries/lists, collections feature, FAQ, appointments, newsletter signup, sustainability, trunk shows, and custom header/footer.

## CI/CD

GitHub Actions runs on every push:
- **Lighthouse CI** — Performance audits on home, product, and collection pages
- **Theme Check** — Shopify theme linting

## Code Style

- **Prettier** with `@shopify/prettier-plugin-liquid`: 120 char width, single quotes (JS), double quotes (Liquid)
- **Theme Check** with `MatchingTranslations` and `TemplateLength` disabled
- All user-facing strings belong in `locales/en.default.json`, not hardcoded in templates