# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static-HTML mockup of the Sumrize dashboard, designed in the TailAdmin style as a reference deliverable for frontend developers. There is no build step, no framework, no backend — just `.html` files loaded directly in the browser (or via `python3 -m http.server` to avoid integrity/CORS quirks on the CDN libs).

Goal: handoff to a frontend dev who will reimplement in their own framework (React/Next/Vue/etc.). Code prioritizes readable markup with section comments and consistent class patterns over production concerns.

## Pages

- `index.html` — Beranda / Your Connectors (filter pills + connector card grid, Add Connector modal)
- `rizz.html` — Ask Rizz chat interface (saved chats sidebar + chat thread + tool-use indicator)
- `invoice.html` — Invoice History (filter bar + table + pagination, demo-toggleable empty state)
- `pricing.html` — Subscribe flow (3 tiers, billing cycle toggle, 3-step checkout modal)
- `referral.html` — Tiered referral program (5 demo stages: empty → progressing → ready → claimed → all-claimed)
- `settings.html` — Account Settings (subscription card + profile form)

All pages share a sidebar + topbar shell. Sidebar collapses on desktop and slides off-canvas on mobile.

## Stack

- **Tailwind v3 via CDN** (`cdn.tailwindcss.com`) — config inlined per page in a `<script>` tag. Brand palette extracted from the live sumrize.me marketing site (not TailAdmin defaults):
  - `brand-50: #eaf4fb` → pale blue wash (sidebar active bg, card footer band)
  - `brand-200: #b3dbff` → light blue accent (most frequent on sumrize.me)
  - `brand-500: #26344a` → navy primary (text on brand-50, badges, focus rings)
  - `brand-700: #1c1f2c` → deep navy
  - `brand-950: #0f0e0e` → near-black
  - `gray-700-950` shifted to Sumrize layered navy: `#3c3f4c` / `#26344a` / `#1c1f2c` / `#0f0e0e`
  - `sumrize: #FF4438` → red, reserved for primary CTAs and destructive actions ONLY (high-attention)
  - `success`/`warning`/`error` are still TailAdmin defaults; can swap success to mint `#3afcb8` later for closer brand match
- **Alpine.js v3 via CDN** — state management. Body-level `x-data` holds shared state (`darkMode`, `lang`, `sidebarToggle`, `profileOpen`, etc.) with `localStorage` persistence on `darkMode` and `lang`.
- **Lucide icons via CDN** — `<i data-lucide="name">` replaced with inline SVG by `lucide.createIcons()` on `DOMContentLoaded`. Style matches TailAdmin (thin outline). Replaced FontAwesome from the original snapshot which couldn't render locally (font files weren't part of the saved page).
- **Outfit font** from Google Fonts.

## Assets

`assets/` contains brand-recognizable icons used as `<img>`:

- `logo-black.png`, `logo-white.png` — Sumrize wordmark, theme-swapped via `dark:hidden` / `hidden dark:block`
- `whatsapp.webp`, `discord.png`, `telegram.png`, `slack.png`, `notion.webp`, `Gmail_Icon.png` — connector logos for cards + modal tiles

Folder was originally named `Sumrize — Because life's too short to scroll group chats_files/` (preserved from the saved-page snapshot) and renamed to `assets/` on 2026-05-13 with all `<img>` references updated in one pass.

## Patterns worth knowing

### State machine + demo toggles

For pages with multiple states (empty/filled/claimed), all states are built into the same file and toggled via Alpine. A "demo control" link in the page footer cycles through stages so reviewers can flip without changing data. See `invoice.html` (`hasSubscription` boolean) and `referral.html` (`demoStage` 0-4 cycler). The toggle area is visually muted (`text-gray-400` + footer position) and commented as `Mockup demo control: remove in production`.

### Bilingual via `t[lang].key`

Body-level `x-data` defines a translation object:

```js
lang: 'en',           // persisted to localStorage
languages: { en: {...}, id: {...} },
t: {
  en: { home: 'Home', rizz: 'Ask Rizz', invoice: 'Invoice', ... },
  id: { home: 'Beranda', rizz: 'Rizz', invoice: 'Invoice', ... }
}
```

Sidebar menu items use `<span x-text="t[lang].home"></span>`. Page-specific one-off strings use inline ternary `x-text="lang === 'id' ? 'X' : 'Y'"`. Switching language via the topbar dropdown updates instantly across the page.

### Sidebar collapse behavior

`sidebarToggle` boolean does double duty:
- mobile (`<lg`): false = drawer off-screen, true = drawer slides in (with backdrop)
- desktop (`lg+`): false = expanded 290px, true = collapsed 90px (icons only + "S" brand mark)

When collapsed on desktop, the brand container becomes `lg:flex-col lg:gap-3` so the S badge stacks above the toggle button.

### Card pattern: brand-bordered "primary" cards

Cards that represent an actionable summary (subscription card, profile form, referral hero) use `border border-brand-500` instead of the default `border-gray-200`. Makes them visually distinct as "the thing to interact with on this page".

### Two-zone card pattern

Connector cards (Beranda) and the profile form (Settings) use a body + footer band layout:
- Body: white/dark surface, regular content with `p-6`
- Footer band: `bg-brand-50` (light blue accent) with the primary action(s)

This pattern is replicated by frontend devs as a reusable `<Card>` component.

## Running locally

```sh
cd "/Users/pajakku/Downloads/sumrize" && python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

Tailwind CDN will log a "should not be used in production" warning. That's expected — for production the frontend dev compiles Tailwind from source.
