# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Papiros Papelería — a single static landing page for a stationery/office-supply store in Usaquén, Bogotá. No build step, no package manager, no framework. Deployed to Vercel as a static site (`.vercel/project.json` links it to project `papiros`).

## Running locally

There is no dev server or build command. Open `index.html` directly in a browser, or serve the directory with any static file server (e.g. `npx serve .`) to avoid `file://` quirks with `fetch`/CORS.

There is no test suite, linter, or formatter configured in this repo.

## Architecture

**`index.html` is the entire application.** All CSS lives in a `<style>` block in `<head>` and all JS lives in a single `<script>` block before `</body>`. It is not split into files and does not import any bundler output.

- `styles.css` is a **leftover/orphaned file** from an earlier scaffold — `index.html` does not link to it (no `<link rel="stylesheet" href="styles.css">` anywhere). Its design tokens (cobalt/blue theme) are stale and superseded by the inline tokens in `index.html` (orange theme, see `:root` near the top of the `<style>` block). Don't assume edits to `styles.css` affect the live page.
- `archivos suministrables/` contains supplier-provided source assets (a copy of `index.html`, product photos) — not part of the served site.

### In-page JS structure (single IIFE, `'use strict'`)

Everything is wired through one self-invoking function with a `boot()` entry point that runs on `DOMContentLoaded`. Each subsystem is initialized via `safe(fn, name)`, which try/catches so one failing subsystem doesn't break the others:

- `initNav` — sticky nav scroll state
- `initMobileMenu` — hamburger menu toggle
- `initReveals` — `IntersectionObserver`-based scroll reveal for `[data-reveal]` elements
- `initContactForm` — builds a `mailto:` link from form fields (no backend/API call)
- `initSmoothScroll` — intercepts in-page `#anchor` clicks for smooth scrolling
- `initCatalogFilters` — wires the search input, price range inputs, and category pills

The product grid is collapsed by default behind a "Ver catálogo completo" toggle (`toggleCatalog()`); selecting a category, typing a search term, or setting a price bound auto-expands it.
- `renderCatalog()` — renders the product grid from the `PRODUCTS` array

There is no cart or checkout flow — the site is a pure catalog/showcase. Ordering happens off-platform via WhatsApp or the third-party delivery apps (Rappi, DiDi Food).

### Product catalog

- `PRODUCTS` is a hardcoded array of objects (`id`, `cat`, `name`, `price`, `img`, `emoji`) grouped by category in comments (papelería, tecnología, juguetería, miscelánea, fiesta, libros, dulcería). `img` URLs point to Unsplash placeholders via the `IMGS` lookup, not real product photos.
- `formatCOP(n)` formats prices as Colombian pesos via `Number.toLocaleString('es-CO')`.
- User-supplied content (product names, etc.) is escaped via `escHTML()` before being injected into `innerHTML` — preserve this when adding new dynamic HTML to avoid XSS.
- Product cards display name and price only (no add-to-cart action).

### Contact flow

There's no payment processor or cart integration. Visitors reach out via WhatsApp deep links (`wa.me/573188682376`) or the contact form, which builds a `mailto:papirospapeleria@gmail.com` link from the form fields. Real order fulfillment happens off-platform (WhatsApp/manual, or via Rappi/DiDi Food), not through this codebase.

### External delivery integrations

The site links out to third-party delivery platforms (Rappi, DiDi Food, Mercado Libre) and WhatsApp — these are plain `<a>` links to external storefronts, not API integrations.

## Git workflow

Every change made in this repo should be committed and pushed to `origin/main` right away (this repo has no branches/PR workflow — Vercel deploys straight from `main`). Don't leave changes uncommitted/unpushed at the end of a task unless the user says otherwise.

## Conventions in the existing code

- CSS class prefix `pp-` (e.g. `.pp-nav`, `.pp-cart-drawer`, `.pp-pay-modal`) — follow this when adding new UI elements.
- CSS custom properties (design tokens) are defined once in `:root` inside the inline `<style>` block — reuse `var(--orange)`, `var(--ink)`, etc. rather than hardcoding colors.
- JS is ES5-style (`var`, function expressions) rather than ES6+ — match this style for consistency within the single script block.
- Spanish is the language for all user-facing copy and most identifiers tied to content (e.g. `nombre`, `mensaje`, `asunto`).
