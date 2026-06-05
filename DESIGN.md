# DESIGN.md

Design and architecture reference for **PETU — People for the Ethical Treatment of Unicorns**, a single-page satirical marketing/parody website.

> This document describes *how the site is built and styled*. The README covers the premise and the day-to-day commands. The voice on the page is deliberately deadpan-satirical; this document is not.

---

## 1. Overview

PETU is a one-page, statically-rendered campaign site for a fictional non-profit that defends the "image rights and iconographic restraint" owed to unicorns. It is built to feel like a polished, kawaii-themed advocacy landing page: soft pastel palette, rounded "blobby" shapes, hand-built SVG mascots, and a layer of GSAP-driven micro-interactions.

There is **no backend, no database, and no real form submission**. Every call-to-action resolves to a satirical modal. The site is content-complete in a single route.

**Goals**
- Read as a credible, charming non-profit landing page at a glance.
- Deliver the satire through tone, copy, and detail — not through breaking the illusion.
- Stay fast, accessible, and fully static (no client framework, minimal JS).
- Be deployable to Cloudflare with one command.

**Non-goals**
- Real donations, auth, persistence, or multi-page routing.
- A component library or design system intended for reuse outside this site.

---

## 2. Tech Stack

| Layer | Choice | Notes |
| :---- | :----- | :---- |
| Framework | [Astro](https://astro.build) `^6.1.9` | Static-first, zero-JS-by-default, `.astro` components. |
| Styling | [Tailwind CSS](https://tailwindcss.com) `^4.2.4` | v4, configured entirely in CSS via `@theme` (no `tailwind.config.js`). Wired in through the `@tailwindcss/vite` plugin. |
| Animation | [GSAP](https://gsap.com) `^3.15.0` | `ScrollTrigger` plugin for scroll-driven reveals and counters. |
| Hosting / Adapter | [Cloudflare](https://developers.cloudflare.com/) via `@astrojs/cloudflare` `^13.2.1` + `wrangler` `^4.85.0` | Server adapter + static assets binding. |
| Build tooling | Vite `^7` (pinned via `overrides`) | Provided by Astro; Tailwind runs as a Vite plugin. |
| Language | TypeScript (strict) | `astro/tsconfigs/strict`. Inline `<script>` blocks are typed. |
| Package manager | Bun | `bun.lock` is the lockfile; scripts are documented with `bun` in the README. |

**Runtime requirement:** Node `>=22.12.0` (see `package.json` `engines`).

### Why this stack
- **Astro** ships the page as HTML/CSS with only the small island of GSAP JS that the interactions actually need. No hydration of a UI framework.
- **Tailwind v4** keeps the entire design language (colors, fonts, radii, shadows) in one `@theme` block in `src/styles/global.css`, so the look is centrally tunable.
- **GSAP** handles all motion in a single `Animations.astro` script rather than scattering animation logic across components.
- **Cloudflare** gives global static delivery; `wrangler.jsonc` binds the built `./dist` as static assets.

---

## 3. Project Structure

```text
petu-site/
├── astro.config.mjs        # site URL, Tailwind Vite plugin, Cloudflare adapter
├── wrangler.jsonc          # Cloudflare Worker + ASSETS binding config
├── tsconfig.json           # extends astro/tsconfigs/strict
├── worker-configuration.d.ts # generated Cloudflare runtime types (wrangler types)
├── package.json            # scripts + deps (Bun lockfile in bun.lock)
├── public/                 # served as-is, not processed
│   ├── favicon.svg         # the unicorn mascot, reused as the logo mark
│   ├── favicon.ico         # legacy fallback
│   └── og-image.svg        # 1200×630 social share card
└── src/
    ├── layouts/
    │   └── Layout.astro     # <html> shell: head, meta, fonts, JSON-LD, <slot/>
    ├── pages/
    │   └── index.astro      # the only route; composes all sections in order
    ├── components/
    │   ├── Navbar.astro      # fixed pill nav + full-screen mobile menu
    │   ├── Hero.astro        # headline, CTAs, animated stat counters
    │   ├── MerchCrisis.astro # "Saturation Audit" room-by-room cards
    │   ├── WhyProtect.astro  # three future-projection cards
    │   ├── GetInvolved.astro # 4-step protocol + donation buttons
    │   ├── Founder.astro      # Mr. Gumpel bio + interactive portrait
    │   ├── Partnership.astro  # PETE cross-promo
    │   ├── Footer.astro       # link columns + the shared satirical modal markup
    │   ├── Animations.astro   # ALL client-side JS (GSAP) for the page
    │   └── svg/               # hand-authored inline SVG components
    │       ├── UnicornMascot.astro  # hero mascot (blinks)
    │       ├── UnicornSad.astro      # crisis mascot (winks)
    │       ├── GumpelPortrait.astro  # founder portrait (reacts)
    │       ├── MerchItem.astro       # 6 merch icons via `variant` prop
    │       ├── Cloud.astro, Rainbow.astro, Sparkle.astro,
    │       └── Star.astro, Heart.astro  # decorative primitives
    └── styles/
        └── global.css        # Tailwind import + @theme design tokens + base styles
```

### Architectural conventions
- **One route.** Astro maps `src/pages/index.astro` to `/`. The page is a thin composition of section components in a fixed top-to-bottom order.
- **Components are sections or SVGs.** Top-level `components/*.astro` are full-width page sections. `components/svg/*.astro` are presentational illustration primitives that accept `class` and (usually) `color` props.
- **All JS lives in one place.** `Animations.astro` is imported once at the bottom of `index.astro`. Components stay markup-only; they expose hooks (IDs, `data-*` attributes, and `petu-*` classes) that the animation script binds to. This keeps the HTML declarative and the behavior centralized.
- **Layout owns the document head.** `Layout.astro` is the only place that renders `<html>/<head>/<body>` and accepts `title` / `description` / `image` props with sensible defaults.

---

## 4. Visual Design Language

The entire design language is defined as tokens in the `@theme` block of `src/styles/global.css`. Tailwind v4 turns each token into a utility (e.g. `--color-petu-pink-500` → `bg-petu-pink-500`, `text-petu-pink-500`).

### 4.1 Color palette

A soft, kawaii pastel system across four hues plus neutrals.

| Token group | Values | Role |
| :---------- | :----- | :--- |
| `petu-cream` | `#fff8f5` | Page background, "paper" surfaces. |
| `petu-pink` `50–600` | `#fff0f6 → #f25aa1` | Primary brand color. CTAs, accents, the hero word "unicorn." |
| `petu-blue` `100–500` | `#e3f1ff → #5a9bff` | Secondary accent. PETE partnership, "blue" projection card. |
| `petu-yellow` `100–400` | `#fff5b8 → #ffd23f` | Tertiary / highlight. Badges, glow behind the mascot. |
| `petu-mint-200` | `#d4f5e0` | Sparing accent. |
| `petu-lavender-200` | `#e8d8ff` | Sparing accent. |
| `petu-ink` | `#3a2a3f` | Primary text + dark footer background. A warm near-black plum. |
| `petu-ink-soft` | `#6b5470` | Secondary / body text. |

Theming notes:
- The dark footer inverts the scheme: `petu-ink` background with `petu-pink-100` text.
- Backgrounds favor soft multi-stop gradients between pink/blue/yellow at low saturation (e.g. the hero `from-petu-pink-50 via-petu-pink-100 to-petu-blue-100`).
- `::selection` is tinted `petu-pink-300`.

### 4.2 Typography

Loaded from Google Fonts in `Layout.astro` with `preconnect` + `display=swap`.

| Token | Stack | Usage |
| :---- | :---- | :---- |
| `--font-display` | `"Fredoka", "Comic Sans MS", system-ui, sans-serif` | All headings (`h1–h4`), nav, numbers, badges. Rounded and friendly. |
| `--font-body` | `"Quicksand", system-ui, sans-serif` | Body copy, paragraphs, the document default. |

- Both families load weights `400;500;600;700`.
- Headings get a tight `letter-spacing: -0.01em` and the display font via a base `h1,h2,h3,h4` rule.
- Display sizes scale aggressively at the `md` breakpoint (e.g. hero `text-5xl md:text-7xl`).

### 4.3 Shape & elevation

| Token | Value | Usage |
| :---- | :---- | :---- |
| `--radius-blob` | `2rem` | Standard card radius. |
| `--radius-blob-lg` | `2.5rem` | Larger feature cards, modal, callouts. |
| `--shadow-kawaii` | `0 8px 0 0 rgba(255,126,184,0.18)` | Signature **flat offset** pink shadow — a solid drop, not a blur. |
| `--shadow-kawaii-blue` | `0 8px 0 0 rgba(126,184,255,0.22)` | Blue variant for secondary/blue elements. |

The flat 8px offset shadow (no blur, no spread) is the defining "sticker"/kawaii look. Cards and buttons sit on a colored cushion rather than a soft drop shadow. Pills (`rounded-full`) are used heavily for nav, badges, buttons, and chips.

### 4.4 Layout system

- **Container:** sections center content with `max-w-6xl mx-auto` (narrower `max-w-5xl`/`max-w-4xl` for the Founder and Partnership sections).
- **Horizontal padding:** `px-6 md:px-10` on sections.
- **Vertical rhythm:** sections use `py-24 md:py-32`.
- **Grids:** card sections use responsive grids (`grid-cols-1 sm:grid-cols-2 lg:grid-cols-3`, etc.).
- **Breakpoints:** Tailwind defaults; `md` (768px) is the primary desktop/mobile switch — the nav, hero layout, and mobile menu all pivot at `md`.

---

## 5. Page Composition

`index.astro` renders the sections in this order inside `Layout`:

1. **`Navbar`** — fixed pill at top.
2. **`<main>`**
   1. `Hero` (`#hero`)
   2. `MerchCrisis` (`#crisis`)
   3. `WhyProtect` (`#why`)
   4. `GetInvolved` (`#get-involved`)
   5. `Founder` (`#founder`)
   6. `Partnership` (`#partnership`)
3. **`Footer`** — link columns + the modal markup.
4. **`Animations`** — invisible; only loads the GSAP script.

Section anchors drive the in-page nav. `html` has `scroll-behavior: smooth` and `scroll-padding-top: 5rem` so anchored sections clear the fixed navbar (smooth scroll is disabled under reduced-motion).

### Section-by-section intent

| Section | ID | Nav label | Purpose / design |
| :------ | :-- | :-------- | :--------------- |
| Hero | `#hero` | (logo) | Headline "Defending the unicorn.", two CTAs, floating clouds/sparkles, the bobbing `UnicornMascot`, and three animated stat counters in a frosted bar pinned to the section bottom. |
| Merch Crisis | `#crisis` | **Audit** | "The Saturation Audit." Six room cards (`MerchItem` icon + count + inventory copy) in a grid, then a gradient callout with the winking `UnicornSad` mascot and a pull quote. |
| Why Protect | `#why` | **Forecast** | "If We Do Not Stop." Three year-stamped projection cards (2031/2034/2040), color-accented pink/blue/yellow, connected by a horizontal gradient timeline line on desktop. |
| Get Involved | `#get-involved` | **Protocol** | "The PETU Protocol." Four numbered steps with badges and action buttons, plus a `$7 / $24 / Other` donation row. All buttons open the satirical modal. |
| Founder | `#founder` | **Founder** | "Mr. Gumpel." Two-column card: interactive `GumpelPortrait` + biography, quote, and fact chips. |
| Partnership | `#partnership` | — | "On the Subject of PETE." Cross-promotes the sibling parody site `thisispete.org` (the only real outbound link). |
| Footer | — | — | Four link columns (brand, sections, contact, legal). Contact/legal links are buttons that open the modal. Credits the author. |

---

## 6. Illustration System (SVG)

All illustrations are **hand-authored inline SVG** Astro components in `src/components/svg/`. No raster images, no icon library, no external sprite. This keeps art crisp at any size, themeable via props, and animatable down to individual elements.

### Conventions
- Every SVG component takes a `class` prop (mapped to the SVG's `class`) so sizing is controlled with Tailwind width utilities (`w-16`, `w-32`, …) at the call site.
- Decorative primitives (`Cloud`, `Sparkle`, `Star`, `Heart`, `Rainbow`) also take a `color` prop with a sensible default, so the same shape recolors per placement.
- Decorative SVGs are marked `aria-hidden="true"`. Meaningful illustrations use `role="img"` + `aria-label` (e.g. the mascots, the favicon).

### Primitives
| Component | viewBox | Props | Notes |
| :-------- | :------ | :---- | :---- |
| `Sparkle` | `0 0 40 40` | `class`, `color` | Four-point sparkle; the workhorse decoration. |
| `Star` | `0 0 40 40` | `class`, `color` | Five-point star. |
| `Heart` | `0 0 40 40` | `class`, `color` | Used in nav CTA, partnership, footer. |
| `Cloud` | `0 0 160 90` | `class`, `color` | Slow-floating background element. |
| `Rainbow` | `0 0 200 110` | `class` | Available primitive. |
| `MerchItem` | `0 0 120 120` | `class`, `variant` | One component, six icons via `variant`: `mug \| shirt \| pillow \| sticker \| lunchbox \| lipbalm`. |

### Character mascots (interactive)
These larger SVGs expose `data-*` hooks that `Animations.astro` targets for hover/click reactions and idle motion:

| Mascot | Mount ID | Interactive hooks | Behavior |
| :----- | :------- | :---------------- | :------- |
| `UnicornMascot` (hero) | `#hero-mascot` | `data-petu-eye="left|right"` | Idle bob + sway; **blinks** on hover/click. |
| `UnicornSad` (crisis) | `#crisis-mascot` | `data-petu-sleepy-right`, `data-petu-mischief` | A sleepy eye opens into a **mischievous wink** on hover; click holds the wink ~1.1s. |
| `GumpelPortrait` (founder) | `#founder-portrait` | `data-gumpel-brows`, `data-gumpel-mouth-default`, `data-gumpel-mouth-grin`, `data-gumpel-sparkle` | Gentle idle sway; on hover the **eyebrows raise, mouth grins, and a tooth sparkle pulses**. Click holds ~1.2s. |

The `favicon.svg` is the same unicorn artwork as the hero mascot and is reused as the navbar logo mark (`<img src="/favicon.svg">`), keeping brand identity consistent across tab, nav, and JSON-LD logo.

---

## 7. Motion & Interaction Design

All client-side behavior is concentrated in **`src/components/Animations.astro`**, a single typed `<script>` (bundled by Astro/Vite) that imports `gsap` and registers `ScrollTrigger`. Components never carry their own JS; they expose class/ID/data hooks and the script binds to them on load.

### 7.1 Reduced-motion as a first-class branch
The very first thing the script reads is `prefers-reduced-motion`. Every animated feature has an explicit reduced-motion path that lands content in its **final visible state** (full opacity, target counter values, split words restored) with no movement. CSS `scroll-behavior` is also reset to `auto` under reduced motion. Accessibility is not an afterthought here — it is a parallel code path throughout.

### 7.2 Animation catalogue

| Effect | Trigger | Mechanism |
| :----- | :------ | :-------- |
| **Navbar entrance** | On load | Pill drops in with `back.out`, then logo / links / CTA stagger in. |
| **Mobile menu** | Toggle button | Full-screen overlay fades in, items stagger; closes on link click, the close button, `Escape`, or backdrop. Locks `body` scroll while open. |
| **Hero entrance choreography** | On load | A single timeline: badge → headline words → subhead words → body → CTAs → mascot pop-in → stats. |
| **Word-split text** | On load (hero) / on scroll (other headings) | `splitIntoWords()` wraps each word in a `.petu-word` span (preserving nested `<strong>`/`<span>`) and scales them in with a bouncy `back.out`. Headings opt in via the `.petu-words` class. |
| **Stat counters** | Hero: in the entrance timeline. Elsewhere: `ScrollTrigger` on enter (once). | `[data-counter]` elements tween `0 → data-target`, formatted with `toLocaleString()` and an optional `data-suffix`. |
| **Idle floats** | Continuous | `.petu-sparkle` (bob + slow 360° spin) and `.petu-float-slow` clouds (gentle horizontal drift). |
| **Section reveals** | On scroll | `.petu-reveal` fades/slides up; `.petu-stagger` containers stagger their `.petu-card` children. |
| **Mascot micro-interactions** | Hover / click | Blink, wink, and the Gumpel reaction described in §6. |
| **Satirical modal** | Click any `[data-petu-action]` | See §7.3. |

### 7.3 The satirical action modal
The modal markup lives in `Footer.astro` (`#petu-modal`) and is driven entirely from `Animations.astro`. It is the payoff for every "action" on the site — donations, pledges, audits, contact links, legal links.

- Any element with `data-petu-action` and a `data-action-label` opens the modal.
- The label is looked up in an `ACTION_MESSAGES` map (title + body of in-character copy). Unknown labels fall back to a generic "Noted, with thanks." message.
- The modal card animates in with `back.out`, locks body scroll, and closes via its buttons, backdrop click, or `Escape`.
- **No network request is ever made.** This is the mechanism that delivers the parody while keeping every CTA "functional."

### 7.4 Animation hook vocabulary
The contract between markup and the animation script:

| Hook | Meaning |
| :--- | :------ |
| `.petu-words` | Heading/paragraph whose words should scale-in (split into `.petu-word`). |
| `.petu-reveal` | Element fades/slides up on scroll. |
| `.petu-stagger` + `.petu-card` | Container whose cards stagger in on scroll. |
| `.petu-sparkle` | Decorative element that bobs and spins forever. |
| `.petu-float-slow` | Element (clouds) that drifts horizontally. |
| `[data-counter][data-target][data-suffix]` | Number that counts up. |
| `[data-petu-action][data-action-label]` | Opens the satirical modal with the mapped message. |
| `[data-petu-modal-close]` | Closes the modal. |
| `#hero-mascot`, `#crisis-mascot`, `#founder-portrait` | Interactive character mounts. |

When adding a section, prefer reusing these hooks over writing new JS.

---

## 8. Accessibility

Accessibility is built into the markup and the motion layer:

- **Reduced motion** is fully honored (§7.1) — no parallax/scroll trickery that can't be disabled.
- **Semantic landmarks:** `<nav aria-label="Primary">`, `<main>`, `<footer>`, real `<h1>–<h4>` hierarchy, `<blockquote>`/`<footer>` for quotes.
- **Mobile menu** is a `role="dialog" aria-modal="true"` with a labelled toggle (`aria-expanded`, `aria-controls`) and Escape-to-close.
- **Modal** uses `role="dialog" aria-modal="true" aria-labelledby`.
- **Decorative SVG** is `aria-hidden`; informative SVG has `role="img"` + `aria-label`. Arrow/emoji glyphs used as decoration are `aria-hidden`.
- **Interactive controls are real `<button>`s** (even the footer "links" that open the modal), not click-handlers on divs.
- **Focus-friendly color contrast** via `petu-ink` text on light surfaces.

---

## 9. SEO, Metadata & Social

`Layout.astro` is comprehensive on metadata for a parody site:

- **Canonical URL** computed from `Astro.site` (`https://petu.info`) + the current path.
- **Open Graph** (type, site_name, title, description, url, image, dimensions `1200×630`, alt, locale) and **Twitter** `summary_large_image` cards, pointing at `public/og-image.svg`.
- **Favicons:** SVG primary, `.ico` fallback, apple-touch, and `mask-icon` (tinted `#ff7eb8`).
- **Theme color** set per light/dark scheme; `color-scheme: light`.
- **JSON-LD structured data** describing the org as a `schema.org/NGO` — name, alternateName, founder (`David Gumpel`), `foundingDate: "1824"`, `foundingLocation: Norway`, and `sameAs: [thisispete.org]`. The in-universe lore is encoded faithfully in the structured data.
- Robots set to `index, follow, max-image-preview:large`; referrer policy `strict-origin-when-cross-origin`.

Title/description/image are props on `Layout`, so a future second page could override them.

---

## 10. Content, Voice & Tone

The design intentionally serves the writing. Tone is **deadpan bureaucratic satire** — a fictional society treating "unicorn iconography saturation" with the gravity of a real advocacy campaign.

Design choices that support the voice:
- **Pseudo-official framing:** badges like "Field Report · 2026", "Forecast · Internal", "Office of the Founder"; "case file restricted, paraphrased with permission"; "All horns reserved."
- **Specific fake data:** counters ("1,400,000,000+ impressions printed daily", "$0.00 royalties since 1824", "47 unicorns per child's bedroom"), itemized room inventories, year-stamped projections.
- **Lore consistency:** founded Norway 1824; founder David Gumpel (glitter auditor, last sighted 1948); archive "above a Norway pâtisserie"; partnership with **PETE** (People for Ethical Treatment of Elves, founded 1823). The JSON-LD and `sameAs` reinforce it.
- **The parody is always disclosed:** the footer states it is "A satirical nonprofit parody. No unicorns (or humans) were harmed."

When editing copy or adding sections, preserve this register: understated, precise, never winking too hard, and always keeping the "non-profit" facade intact (CTAs open the modal rather than admitting nothing happens).

---

## 11. Build, Deploy & Tooling

### Scripts (`package.json`)
| Command | Action |
| :------ | :----- |
| `bun dev` | Astro dev server at `localhost:4321`. |
| `bun build` | Builds to `./dist/`. |
| `bun preview` | Serves the production build locally. |
| `bun astro …` | Astro CLI passthrough. |
| `bun run generate-types` | `wrangler types` → regenerates `worker-configuration.d.ts`. |
| `bun deploy` | `astro build && wrangler types && wrangler deploy`. |

### Configuration
- **`astro.config.mjs`** — sets `site: "https://petu.info"`, registers Tailwind as a Vite plugin, and uses the `cloudflare()` adapter.
- **`wrangler.jsonc`** — Worker name `petu-site`, `main` pointed at the Astro Cloudflare server entrypoint, `assets` binding `ASSETS` → `./dist`, `compatibility_date` `2026-04-26`, `global_fetch_strictly_public` flag, observability disabled.
- **`tsconfig.json`** — extends Astro's strict config; includes generated `.astro/types.d.ts` and `worker-configuration.d.ts`; excludes `dist`.
- **`worker-configuration.d.ts`** — generated by `wrangler types`; do not edit by hand.

### Ignored / generated (`.gitignore`)
`dist/` (build output), `.astro/` (generated types), `node_modules/`, env files, and OS/editor cruft are not committed.

---

## 12. Conventions & Extension Guide

Patterns to follow when working in this repo:

1. **New section** → create `src/components/<Name>.astro` as a full-width section with an `id`, import it into `index.astro`, and place it in the `<main>` order. Add a `navLinks` entry in `Navbar.astro` if it should appear in the nav (both desktop and mobile lists render from that one array).
2. **Reuse the motion hooks** (`.petu-reveal`, `.petu-stagger`/`.petu-card`, `.petu-words`, `[data-counter]`) rather than writing new JS. They already have reduced-motion fallbacks.
3. **New CTA** → render a `<button data-petu-action data-action-label="…">` and add a matching entry to `ACTION_MESSAGES` in `Animations.astro`. Never wire a real submission.
4. **New illustration** → add an inline SVG component under `components/svg/` with a `class` prop (and a `color` prop if it's a recolorable decoration). Mark decorative art `aria-hidden`.
5. **Colors/typography/shape** → change tokens in the `@theme` block of `global.css`; don't hardcode hex values in components.
6. **Keep JS centralized** in `Animations.astro`; keep components markup-only.
7. **Honor reduced motion** for anything new that moves — provide a final-state fallback.
8. **Keep the parody disclosed** and avoid real data collection; the site collects and stores nothing.

---

*Maintained alongside the codebase. If the structure here drifts from what's in `src/`, the code wins — update this doc.*
