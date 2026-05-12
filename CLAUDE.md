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

- **Tailwind v3 via CDN** (`cdn.tailwindcss.com`) — config inlined per page in a `<script>` tag. Brand tokens: `brand-500: #465fff` (TailAdmin indigo), `sumrize: #FF4438` (Sumrize red), Untitled UI gray palette, plus `success`/`warning`/`error` semantic colors.
- **Alpine.js v3 via CDN** — state management. Body-level `x-data` holds shared state (`darkMode`, `lang`, `sidebarToggle`, `profileOpen`, etc.) with `localStorage` persistence on `darkMode` and `lang`.
- **Lucide icons via CDN** — `<i data-lucide="name">` replaced with inline SVG by `lucide.createIcons()` on `DOMContentLoaded`. Style matches TailAdmin (thin outline). Replaced FontAwesome from the original snapshot which couldn't render locally (font files weren't part of the saved page).
- **Outfit font** from Google Fonts.

## Assets

`Sumrize — Because life's too short to scroll group chats_files/` contains brand-recognizable icons used as `<img>`:

- `logo-black.png`, `logo-white.png` — Sumrize wordmark, theme-swapped via `dark:hidden` / `hidden dark:block`
- `whatsapp.webp`, `discord.png`, `telegram.png`, `slack.png`, `notion.webp`, `Gmail_Icon.png` — connector logos for cards + modal tiles

The folder name has a space, an em-dash (`—` U+2014), and a curly apostrophe (`'` U+2019). Quote it as `Sumrize*chats_files/` when shell-globbing, or quote the full path with double quotes. Renaming the folder requires updating ~50 `<img src=...>` references across all 6 pages.

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
