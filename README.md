# The Fall Bride — Custom Shopify Theme

A custom Shopify theme for [The Fall Bride](https://london.thefallbride.com), a bridal store from London, built on a fork of Shopify's [Dawn](https://github.com/Shopify/dawn) v15.4.0.

[Project Overview](#project-overview) |
[Stores & Branches](#stores--branches) |
[Getting Started](#getting-started) |
[Branch Workflow](#branch-workflow) |
[Staying Up to Date with Dawn](#staying-up-to-date-with-dawn) |
[Architecture](#architecture) |
[Developer Tools](#developer-tools) |
[Code Style](#code-style)

---

## Project Overview

The Fall Bride theme follows Dawn's web-native, HTML-first, JavaScript-only-as-needed philosophy:

* **Server-rendered:** All HTML is rendered by Shopify via Liquid. JavaScript is used sparingly for progressive enhancement only.
* **Lean, fast, and reliable:** No render-blocking JS, no DOM manipulation before user input, no frameworks.
* **Functional, not pixel-perfect:** Semantic markup and progressive enhancement ensure the theme works across all browsers.

### Remotes

| Remote | URL |
|--------|-----|
| `origin` | `git@github.com:ahref13/dawn.git` |
| `upstream` | `https://github.com/Shopify/dawn.git` |

---

## Stores & Branches

The theme serves two separate Shopify stores, each connected to its own branch:

| Branch | Store | Live URL |
|--------|-------|----------|
| `london` | The Fall Bride London | [london.thefallbride.com](https://london.thefallbride.com) |
| `newyork` | The Fall Bride New York | [newyork.thefallbride.com](https://newyork.thefallbride.com) |
| `main` | Development / shared base | Dev store: `the-fall-dev.myshopify.com` |

The `main` branch is the shared development base. The `london` and `newyork` branches are the production branches deployed to each store.

---

## Getting Started

### Prerequisites

- [Shopify CLI](https://shopify.dev/docs/themes/tools/cli)
- [Node.js](https://nodejs.org/) (for Tailwind CSS compilation)

### Commands

| Command | Purpose |
|---------|---------|
| `npm run build:css` | Compile Tailwind CSS in watch mode (`assets/app.css` → `assets/output.css`) |
| `shopify theme dev --store the-fall-dev.myshopify.com` | Start local dev server (main/dev) |
| `shopify theme check` | Lint theme (Liquid, performance, best practices) |
| `shopify theme push --development` | Push to development theme on store |

---

## Branch Workflow

### Golden Rules

1. **Never work directly on `london` or `newyork` branches.** Always develop on `main` or a feature branch off `main`.
2. **Never merge `london` into `newyork` or vice versa.** The two store branches should never exchange changes directly — this prevents store-specific config and content from leaking between stores.
3. **All shared code flows through `main`.** Both store branches receive updates from `main`, never from each other.

### Developing a Shared Feature (applies to both stores)

```
main ← feature/my-feature
```

1. Create a feature branch from `main`:
   ```sh
   git checkout main
   git pull origin main
   git checkout -b feature/my-feature
   ```
2. Develop and test your changes.
3. Merge (or PR) into `main`:
   ```sh
   git checkout main
   git merge feature/my-feature
   git push origin main
   ```
4. Then propagate to both store branches:
   ```sh
   git checkout london
   git pull origin london
   git merge main
   git push origin london

   git checkout newyork
   git pull origin newyork
   git merge main
   git push origin newyork
   ```

### Developing a Store-Specific Feature (one store only)

```
london ← feature/london-specific-thing
```

1. Branch off the target store branch:
   ```sh
   git checkout london
   git pull origin london
   git checkout -b feature/london-specific-thing
   ```
2. Develop and test.
3. Merge back into **only** that store branch:
   ```sh
   git checkout london
   git merge feature/london-specific-thing
   git push origin london
   ```
4. **Do NOT merge this into `main` or `newyork`.**

### Quick Reference: What Goes Where

| Change type | Develop on | Merge into |
|-------------|-----------|------------|
| Shared feature / bug fix | `feature/*` off `main` | `main` → `london` + `newyork` |
| London-only change | `feature/*` off `london` | `london` only |
| New York-only change | `feature/*` off `newyork` | `newyork` only |
| Dawn upstream update | `main` (merge from upstream) | `main` → `london` + `newyork` |

### Handling Merge Conflicts

When merging `main` into a store branch, conflicts will typically appear in:
- `config/settings_data.json` — Each store has its own customizer settings. **Always keep the store branch version** for this file.
- `templates/*.json` — Template configurations may differ per store. Review carefully and keep the store-specific version when in doubt.

---

## Staying Up to Date with Dawn

This theme tracks Shopify's Dawn as an `upstream` remote so we can pull in bug fixes and new features.

### Setup (one-time)

Verify you have both remotes:
```sh
git remote -v
```

If `upstream` is missing, add it:
```sh
git remote add upstream https://github.com/Shopify/dawn.git
```

### Pulling Dawn Updates

1. Fetch and merge upstream into `main`:
   ```sh
   git checkout main
   git fetch upstream
   git merge upstream/main
   ```
2. Resolve any merge conflicts. Our custom sections (`tf-*` prefix) should rarely conflict since they don't exist in Dawn.
3. Test thoroughly, then push:
   ```sh
   git push origin main
   ```
4. Propagate to both store branches:
   ```sh
   git checkout london
   git pull origin london
   git merge main
   git push origin london

   git checkout newyork
   git pull origin newyork
   git merge main
   git push origin newyork
   ```

### Tips for Dawn Merges

- Dawn's `config/settings_schema.json` changes frequently — review these carefully as our custom settings live here too.
- New Dawn sections/snippets won't conflict with our `tf-*` prefixed files.
- Always run `shopify theme check` after merging to catch any regressions.

---

## Architecture

### Directory Layout

| Directory | Purpose |
|-----------|---------|
| `layout/` | Root layouts (`theme.liquid` is the main entry point) |
| `sections/` | Modular page sections (Online Store 2.0). Custom sections use `tf-` prefix |
| `snippets/` | Reusable Liquid partials included via `{% render %}` |
| `templates/` | JSON templates composing sections into pages. Includes `customers/` and `metaobject/` subdirectories |
| `assets/` | CSS and JS files. `app.css` is the Tailwind input; `output.css` is generated (**do not edit directly**) |
| `config/` | `settings_schema.json` (theme customizer definition) and `settings_data.json` (current values) |
| `locales/` | 53+ translation files; `en.default.json` is the source of truth |

### Styling

- **Tailwind CSS v4** compiled via `@tailwindcss/cli`. Config scans `layout/`, `sections/`, `snippets/`, and `templates/`
- **CSS custom properties** for color schemes, defined in `settings_schema.json` and applied in `theme.liquid`
- **Component CSS** files (`tf-*.css`) for complex custom sections

### JavaScript

Vanilla JS only — no frameworks. Key files:

| File | Purpose |
|------|---------|
| `global.js` | Core utilities (`HTMLUpdateUtility`, `SectionId`, focusable element helpers) |
| `pubsub.js` | Pub/sub event system for cross-component communication |
| `product-form.js` / `product-info.js` | Product page logic |
| `cart-drawer.js` | AJAX cart drawer |
| `facets.js` | Collection filtering |
| `animations.js` | Scroll-triggered animations (toggled via theme settings) |

### Custom Sections (`tf-*`)

The Fall adds 24 custom sections for: hero animations, designer carousels/galleries/lists, collections feature, FAQ, appointments, newsletter signup, sustainability, trunk shows, and custom header/footer.

---

## Developer Tools

### Shopify CLI

[Shopify CLI](https://github.com/Shopify/shopify-cli) is used for local development, theme pushing, and store management. Follow the [quick start guide](https://shopify.dev/docs/themes/tools/cli) to get started.

### Theme Check

[Theme Check](https://github.com/shopify/theme-check) validates and lints Shopify themes. It's included in the [VS Code extensions list](/.vscode/extensions.json) and can be run from the terminal:

```bash
shopify theme check
```

### Continuous Integration

GitHub Actions runs on every push:
- **[Lighthouse CI](https://github.com/Shopify/lighthouse-ci-action)** — Performance audits on home, product, and collection pages
- **[Theme Check](https://github.com/Shopify/theme-check-action)** — Shopify theme linting

---

## Code Style

- **Prettier** with `@shopify/prettier-plugin-liquid`: 120 char width, single quotes (JS), double quotes (Liquid)
- **Theme Check** with `MatchingTranslations` and `TemplateLength` disabled
- All user-facing strings belong in `locales/en.default.json`, not hardcoded in templates

---

## Dawn Base Theme

This theme is forked from [Shopify/Dawn](https://github.com/Shopify/dawn) v15.4.0. Dawn is Shopify's first source-available theme, following a web-native, HTML-first approach. See the [Dawn contribution guide](https://github.com/Shopify/dawn/blob/main/.github/CONTRIBUTING.md#theme-code-principles) for its core code principles.

## License

Copyright (c) 2021-present Shopify Inc. See [LICENSE](/LICENSE.md) for further details.
