# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **static, pre-built mirror** of a Lovable.app landing page / quiz funnel for a Brazilian weight-loss product ("Truque da Gelatina Mounjaro"). It is not a source project — there is no `package.json`, no build tooling, and no dev server. The site is served as plain files (`index.html` + hashed Vite bundle + images).

The parent folder (`ESTRUTURA [SITE]`) is the site structure used for ad campaigns; `QUIZ/` is one funnel step (the quiz landing). Sibling folders at the `OFERTAS/GELATINA MOUNJARO/` level likely contain other pages of the same offer.

## Layout

- [index.html](index.html) — single entry point. Hosts the `<div id="root">` the React bundle mounts into, plus a large amount of **inline infrastructure scripts** (see below).
- [js/index-Bqd6vyIR.js](js/index-Bqd6vyIR.js) — minified, hash-named Vite/React production bundle (~490 KB, single line). **Cannot be edited as source code** — it is compiled output. However, **hardcoded strings and numeric references CAN be found and replaced** (e.g. prices, URLs, hardcoded screen numbers). The bundle contains 26 quiz screens (cases 1–26 in a `switch(t)` statement). Each screen is marked with a `/* TELA NN */` comment for easy location (use `Ctrl+F "TELA NN"` to navigate).
- [css/index-CyrDudUx.css](css/index-CyrDudUx.css) — single-line minified Tailwind build paired with the JS bundle (same hash scheme).
- [js/pixel.js](js/pixel.js) — local copy of Utmify tracking pixel (also loaded remotely from `cdn.utmify.com.br` via a script injected in `<head>`; pixel id `698d0b2abc83efb32f889a0d`).
- [js/~flock.js](js/~flock.js) — analytics loader, proxied through `/~api/analytics`.
- [assets/](assets/) — hash-named images referenced by the React bundle (transformation photos, body silhouettes, logos, etc.). Filenames match Vite's asset hashing; renaming them will break the bundle.
- [images/](images/) — OG/social preview image only.

## The inline scripts in index.html — read before editing

`index.html` is mostly **not** page markup. The `<body>` is empty except for `#root`. What it does contain, and why, matters if you touch the file:

1. **CDN/proxy rewriter** (two blocks: `__sys_proxy_guard` at the top of `<head>`, and `__cdn_routing_core` at the bottom of `<body>`). These monkey-patch `fetch`, `XMLHttpRequest.open`, `WebSocket`, `EventSource`, `Worker`, `navigator.sendBeacon`, `document.createElement`, `CSSStyleSheet.insertRule`, `HTMLLinkElement.href`, `Location.prototype.href/hostname/host/origin`, and `window.open` to rewrite the origin `gelatinmounjaro.lovable.app` → `gelatinmou2plg.arktrix.com`. They also inject an `<script type="importmap">` for the same remap and block `about:blank` navigations (used by anti-cloning scripts on the original host). This is the mechanism that lets the mirrored bundle still call back to a working proxy backend. **Do not remove or reorder these blocks** — the React bundle will break without them.
2. **Arktrix visibility fallback `<style>`** — forces animated builder elements (Tilda, AOS, Elementor, Tailwind opacity-0 + animation-delay, etc.) to become visible after 1–2s in case the builder JS fails to run. Safety net for the clone.
3. **Supabase presence tracker** — sends `ping`/`leave` heartbeats every 90s to `https://hwlevhfqsmyssdpktyhl.supabase.co/functions/v1/track-presence` with a hard-coded `OFFER_ID` (`68248ef7-bbe6-452d-a1d5-0eb0fc2cd86d`) and a `SESSION_ID` persisted as cookie `_csid`. Powers the "X people viewing now" style counters in the original funnel.
4. **Parent-frame nav relay** — posts `{t:'sys-nav', u: location.href}` to `window.parent` on every history change, for when the page is embedded in a preview/builder iframe.

The `og:image` `<meta>` deliberately points at a local file in `images/`; the Twitter variant points at the original Lovable R2 bucket. Keep both unless you are replacing the social preview.

## Working on this codebase

- **No build, no tests, no lint.** There is nothing to run. To view the page, open `index.html` directly in a browser or serve the folder with any static server (e.g. `python -m http.server` from this directory).
- **Content edits** that live in the React bundle (quiz questions, copy, prices, CTAs) are **not editable here** — they're minified inside `js/index-Bqd6vyIR.js`. If the user asks to change such content, tell them it has to be changed in the upstream Lovable project and re-exported, or done via a post-load DOM patch script injected into `index.html`.
- **Safe edits in `index.html`**: `<title>`, meta tags, the tracking pixel id, the Supabase `OFFER_ID`/presence URL, the proxy origin/proxy pair (`O`/`P` and `ORIGIN`/`PROXY` constants — they must match in both script blocks), and the OG image path. When changing the proxy pair, update it in **all** locations (top guard, bottom core, importmap).
- **Asset swaps**: replacing images in `assets/` requires keeping the exact hashed filename, since the bundle references them by that name. To use a new image, overwrite the file in place rather than adding a new one with a different name.
- **Pixel / analytics**: the Utmify pixel is loaded twice (once from CDN, once from local `js/pixel.js`) — this is intentional redundancy from the clone. `window.pixelId` is set inline before either loads.

## Quiz structure (26 telas)

The quiz is a linear 26-screen flow (cases 1–26 in `js/index-Bqd6vyIR.js`). Screen state is tracked by variable `t` (first screen = 1, last = 26). Navigation is always `t+1` (no jumps). All screens are findable by searching `/* TELA NN */` in the bundle.

**Key screens:**
- Screen 1: Age selection ("Qual a sua idade?")
- Screen 5: User name input
- Screen 10: Current weight (kg) — critical for BMI calculation downstream
- Screen 11: Height (cm) — critical for BMI
- Screen 12: Target weight (kg) — used in final offer ("Você vai perder X kg")
- Screen 19: **VSL 1** (video embed `id="panda-player"`, PandaVideo player)
- Screen 20: Metabolic analysis result (uses BMI from screens 10+11)
- Screen 24: **VSL 2** (video embed `id="panda-player-2"`, PandaVideo player)
- Screen 26: Final offer + price breakdown (`MM` component, `weightLoss:a-f`)

**Quick edits in the bundle:**
- Search `/* TELA 19 */` to find VSL 1 URL (currently `player-vz-db796669-357.tv.pandavideo.com.br/embed/?v=f3c1f761-ad93-4572-9e2e-837ef824fddf`)
- Search `/* TELA 24 */` to find VSL 2 URL (currently `player-vz-db796669-357.tv.pandavideo.com.br/embed/?v=dd33b44d-6ccd-4130-b143-0536632bece3`)
- Search `6x de R$` to find payment plan (currently `6x de R$4,50`)

**Hardcoded screen numbers (outside the switch):**
If you renumber screens, update these 6 references:
- `t/26*100` — progress bar denominator (currently 26 because there are 26 screens)
- `if(t!==26)return` — countdown only runs on final screen
- `t!==25&&t!==26` — hide back button on last 2 screens
- `t!==26` — hide progress bar on final screen (appears 1x)
- `t===26` — show countdown timer on final screen

## DevNav (localhost development nudge)

A floating ⚙ button appears in bottom-right corner if `window.location.hostname === 'localhost'` or `127.0.0.1`:
- Click → opens a panel to jump directly to any screen (1–26)
- Useful for testing. Disappears on production/non-localhost domains.
- Injected at end of root component (`<div id="root">` parent), minimal footprint.

## Backups

- `index-Bqd6vyIR.backup.js` — original, untouched
- `index-Bqd6vyIR.v1.js` — v1: `/* TELA */` comments + price `6x de R$4,50` + DevNav added
- `index-Bqd6vyIR.v2.js` — v2: v1 + VSL 1&2 URLs updated
- `index-Bqd6vyIR.v3.js` — v3: snapshot before reordering
- `index-Bqd6vyIR.v4.js` — v4: snapshot after 26-screen reordering
- `index-Bqd6vyIR.js` — current live version
