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
- `initCart` — cart drawer open/close, wires checkout button to the payment modal
- `initPayment` — multi-step payment modal (4 steps, tracked via `payStep`/`payMethod`)
- `initReveals` — `IntersectionObserver`-based scroll reveal for `[data-reveal]` elements
- `initContactForm` — builds a `mailto:` link from form fields (no backend/API call)
- `initSmoothScroll` — intercepts in-page `#anchor` clicks for smooth scrolling
- `renderCatalog('all')` — renders the product grid from the `PRODUCTS` array

### Product catalog & cart

- `PRODUCTS` is a hardcoded array of objects (`id`, `cat`, `name`, `price`, `img`, `emoji`) grouped by category in comments (papelería, tecnología, juguetería, miscelánea, fiesta, libros, dulcería). `img` URLs point to Unsplash placeholders via the `IMGS` lookup, not real product photos.
- Cart state (`cart`, array of `{id, qty}`) is persisted to `localStorage` under key `pp_cart`. There is no backend — everything is client-side.
- `formatCOP(n)` formats prices as Colombian pesos via `Number.toLocaleString('es-CO')`.
- Inline `onclick` handlers (e.g. `ppChangeQty`, `ppRemoveItem`) call functions exposed on `window` — keep that pattern when touching cart-item rendering (`renderCartItems`), since markup is built via string concatenation (`innerHTML`), not templating.
- User-supplied content (product names, etc.) is escaped via `escHTML()` before being injected into `innerHTML` — preserve this when adding new dynamic HTML to avoid XSS.

### Checkout flow

There's no payment processor integration. The payment modal walks the user through steps and ends by deep-linking to WhatsApp (`wa.me/573188682376`) with a pre-filled order message, or via `mailto:papirospapeleria@gmail.com` for the contact form. Real order fulfillment happens off-platform (WhatsApp/manual), not through this codebase.

### External delivery integrations

The site links out to third-party delivery platforms (Rappi, DiDi Food, Mercado Libre) and WhatsApp — these are plain `<a>` links to external storefronts, not API integrations.

## Conventions in the existing code

- CSS class prefix `pp-` (e.g. `.pp-nav`, `.pp-cart-drawer`, `.pp-pay-modal`) — follow this when adding new UI elements.
- CSS custom properties (design tokens) are defined once in `:root` inside the inline `<style>` block — reuse `var(--orange)`, `var(--ink)`, etc. rather than hardcoding colors.
- JS is ES5-style (`var`, function expressions) rather than ES6+ — match this style for consistency within the single script block.
- Spanish is the language for all user-facing copy and most identifiers tied to content (e.g. `nombre`, `mensaje`, `asunto`).
