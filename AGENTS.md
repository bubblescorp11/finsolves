# AGENTS.md — Finsolves Project Instructions

> **This file is the single source of truth for all future code changes, AI agent instructions, and developer onboarding.**
> Every decision made during the build of this project is documented here.
> Read this entire file before making any changes to the codebase.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack & Exact Versions](#2-tech-stack--exact-versions)
3. [Folder Structure](#3-folder-structure)
4. [Critical Styling Rules](#4-critical-styling-rules)
5. [Tailwind v4 Architecture — Must Read](#5-tailwind-v4-architecture--must-read)
6. [Color System & Design Tokens](#6-color-system--design-tokens)
7. [Typography](#7-typography)
8. [Pages & Routing](#8-pages--routing)
9. [Component Reference](#9-component-reference)
10. [Design Reference & UI Rules](#10-design-reference--ui-rules)
11. [Security — CVEs Fixed & Version Constraints](#11-security--cves-fixed--version-constraints)
12. [Dependency Rules](#12-dependency-rules)
13. [Configuration Files](#13-configuration-files)
14. [Do's and Don'ts](#14-dos-and-donts)
15. [Running the Project](#15-running-the-project)
16. [Change History](#16-change-history)

---

## 1. Project Overview

**Company:** Finsolves
**Type:** Static marketing website (demo project)
**Domain:** https://finsolves.com
**Business:** Finsolves is a financial services company that offers SAP Group Reporting solutions. They help organizations streamline financial consolidation, ensure compliance, and deliver trusted insights across global operations.

**Website purpose:** Premium enterprise SaaS-style marketing site that showcases Finsolves' SAP Group Reporting expertise. The design is modelled after a high-end fintech SaaS product (FinCore reference design) featuring a live-looking SAP dashboard mockup as the hero visual.

---

## 2. Tech Stack & Exact Versions

### Production Dependencies

| Package | Version | Notes |
|---|---|---|
| `astro` | `^6.3.7` | **Minimum 6.3.7** — earlier versions have active CVEs (see §11) |

### Dev Dependencies

| Package | Version | Notes |
|---|---|---|
| `tailwindcss` | `^4.1.0` | Tailwind v4 — architecture is completely different from v3 |
| `@tailwindcss/vite` | `^4.1.0` | Vite plugin replacing `@astrojs/tailwind`. Must match tailwindcss version |
| `typescript` | `^5.7.0` | Required for Astro 6 strict mode |

### Removed / Banned Packages

| Package | Reason |
|---|---|
| `@astrojs/tailwind` | Deprecated — replaced by `@tailwindcss/vite` in Tailwind v4 |
| `tailwindcss@3.x` | Old version — replaced by v4 |

### package.json (canonical)

```json
{
  "name": "finsolves",
  "type": "module",
  "version": "1.0.0",
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "astro": "astro"
  },
  "dependencies": {
    "astro": "^6.3.7"
  },
  "devDependencies": {
    "@tailwindcss/vite": "^4.1.0",
    "tailwindcss": "^4.1.0",
    "typescript": "^5.7.0"
  }
}
```

---

## 3. Folder Structure

```
finsolves/
├── AGENTS.md                         ← this file
├── astro.config.mjs                  ← Astro + Vite config
├── tsconfig.json                     ← TypeScript config
├── package.json
├── public/
│   └── favicon.svg                   ← SVG favicon (bar chart icon, blue)
└── src/
    ├── styles/
    │   └── global.css                ← Tailwind v4 entrypoint ONLY (see §5)
    ├── layouts/
    │   └── Layout.astro              ← Base HTML shell, SEO meta, font imports
    ├── components/
    │   ├── Navbar.astro              ← Sticky navbar with mobile hamburger menu
    │   ├── Hero.astro                ← Hero section (text + dashboard mockup)
    │   ├── DashboardMockup.astro     ← SAP Group Reporting dashboard UI (SVG/HTML)
    │   ├── WhyChoose.astro           ← Feature cards section
    │   ├── TrustedBy.astro           ← Client logo strip
    │   ├── CTA.astro                 ← Dark gradient call-to-action banner
    │   └── Footer.astro              ← Site footer with links and contact info
    └── pages/
        ├── index.astro               ← Home page
        ├── services.astro            ← Services grid page
        ├── about.astro               ← About us, stats, values, team
        └── contact.astro             ← Contact page with validated form
```

> **Rule:** Never create files outside this structure without updating this document.

---

## 4. Critical Styling Rules

These rules were set by the project owner and must never be violated:

### ✅ ALWAYS use Tailwind utility classes

Every style must be expressed as Tailwind utility classes directly on HTML elements.

```astro
<!-- CORRECT -->
<div class="flex items-center gap-4 px-6 py-3 bg-primary-600 text-white rounded-lg">

<!-- WRONG -->
<div class="my-custom-card">
```

### ❌ NEVER write custom CSS

Do not write CSS rules, custom classes, `style` blocks, or `<style>` tags anywhere in any `.astro`, `.ts`, `.js`, or `.html` file.

```astro
<!-- BANNED — never do this -->
<style>
  .my-card { padding: 1rem; background: blue; }
</style>

<!-- BANNED — never do this -->
<div style="padding: 1rem; background: blue;">
```

### ⚠️ The ONE exception — global.css

`src/styles/global.css` exists **only** because Tailwind v4 requires it as its CSS entrypoint. It contains:
- `@import "tailwindcss"` — loads the framework
- `@theme {}` — defines custom design tokens (colors, fonts, shadows)

**This file must not grow.** Do not add any CSS rules, custom classes, or component styles to it. It is infrastructure, not styling. All visual styling lives in Tailwind classes on components.

### ❌ NEVER add a second CSS file

There is one CSS file in the entire project. Do not create `styles/components.css`, `styles/custom.css`, or any other stylesheet.

---

## 5. Tailwind v4 Architecture — Must Read

Tailwind v4 is a complete rewrite from v3. The integration model is entirely different.

### How Tailwind v4 is wired up

```
src/styles/global.css
  └── @import "tailwindcss"         ← loads Tailwind
  └── @theme { ... }                ← custom tokens

astro.config.mjs
  └── vite.plugins: [tailwindcss()] ← @tailwindcss/vite processes the CSS

Layout.astro
  └── import '../styles/global.css' ← single import, applies globally
```

### Key differences from Tailwind v3

| Concern | Tailwind v3 | Tailwind v4 |
|---|---|---|
| Config file | `tailwind.config.mjs` | **Does not exist** |
| Integration | `@astrojs/tailwind` | `@tailwindcss/vite` plugin |
| Custom tokens | `theme.extend` in config | `@theme {}` in CSS file |
| Gradient syntax | `bg-gradient-to-br` | `bg-linear-to-br` |
| CSS entrypoint | Optional | **Required** |

### Gradient utility rename (breaking change)

All gradient direction classes were renamed in v4:

```
bg-gradient-to-t   →   bg-linear-to-t
bg-gradient-to-tr  →   bg-linear-to-tr
bg-gradient-to-r   →   bg-linear-to-r
bg-gradient-to-br  →   bg-linear-to-br
bg-gradient-to-b   →   bg-linear-to-b
bg-gradient-to-bl  →   bg-linear-to-bl
bg-gradient-to-l   →   bg-linear-to-l
bg-gradient-to-tl  →   bg-linear-to-tl
```

> **If you add a gradient anywhere, always use `bg-linear-to-*` not `bg-gradient-to-*`.**

---

## 6. Color System & Design Tokens

All colors are defined in `src/styles/global.css` under `@theme {}` and are available as Tailwind utilities.

### Primary (Blue — main brand color)

| Token | Hex | Usage |
|---|---|---|
| `primary-50` | `#eff6ff` | Hover backgrounds, icon backgrounds |
| `primary-100` | `#dbeafe` | Light backgrounds |
| `primary-200` | `#bfdbfe` | Borders on hover |
| `primary-300` | `#93c5fd` | Focus rings |
| `primary-400` | `#60a5fa` | Logo accent bar, secondary elements |
| `primary-500` | `#3b82f6` | Charts, decorative elements |
| `primary-600` | `#2563eb` | **Primary brand color — buttons, links, active states** |
| `primary-700` | `#1d4ed8` | Hover state for primary buttons |
| `primary-800` | `#1e40af` | Active/pressed state |
| `primary-900` | `#1e3a8a` | Deep accents |

### Navy (Dark backgrounds)

| Token | Hex | Usage |
|---|---|---|
| `navy-700` | `#253858` | Secondary dark text |
| `navy-800` | `#1e293b` | CTA section background, footer |
| `navy-900` | `#0f172a` | Headings, primary dark text |

### Slate (Neutral grays — from Tailwind defaults)

Use Tailwind's built-in `slate-*` scale for body text, borders, and backgrounds. Key values:
- `slate-50` — page background tints
- `slate-100` — borders, dividers
- `slate-400` — placeholder text, muted labels
- `slate-500` — body copy, descriptions
- `slate-600` — secondary text, nav links
- `slate-700` — dark body text
- `slate-800` — headings
- `slate-900` — darkest text

### Custom Shadows

These are defined in `@theme {}` and available as Tailwind utilities:

| Class | Usage |
|---|---|
| `shadow-card` | Default card border shadow |
| `shadow-card-hover` | Card shadow on hover (blue-tinted) |
| `shadow-dashboard` | Hero dashboard mockup drop shadow |

---

## 7. Typography

### Fonts

Two fonts are loaded via Google Fonts in `Layout.astro`:

| Font | Variable | Usage |
|---|---|---|
| **Sora** | `font-display` | All headings (`h1`–`h4`), logo, card titles |
| **DM Sans** | `font-sans` (default) | All body text, navigation, buttons, labels |

### Font weight conventions

| Weight | Class | Usage |
|---|---|---|
| 300 | `font-light` | Large decorative numbers |
| 400 | `font-normal` | Body copy |
| 500 | `font-medium` | Nav links |
| 600 | `font-semibold` | Buttons, card titles, labels |
| 700 | `font-bold` | Section headings (h2, h3) |
| 800 | `font-extrabold` | Hero h1, major display headings |

---

## 8. Pages & Routing

Astro uses file-based routing. All pages are in `src/pages/`.

| File | Route | Description |
|---|---|---|
| `index.astro` | `/` | Home — Hero, WhyChoose, TrustedBy, CTA sections |
| `services.astro` | `/services` | Services grid — 6 service cards |
| `about.astro` | `/about` | About — stats, core values, leadership team |
| `contact.astro` | `/contact` | Contact form + contact info cards |

### Navigation links

The navbar contains exactly these four links in this order:

1. Home → `/`
2. Services → `/services`
3. About Us → `/about`
4. Contact → `/contact`

> **Do not add new nav links without updating `Navbar.astro` and this document.**

### Active state logic

The navbar uses `Astro.url.pathname` to detect the active page:

```astro
const isActive = currentPath === link.href ||
  (link.href !== '/' && currentPath.startsWith(link.href));
```

Active links receive: `text-primary-600 bg-primary-50`
Inactive links receive: `text-slate-600 hover:text-primary-600 hover:bg-slate-50`

---

## 9. Component Reference

### `Layout.astro`
- Wraps every page
- Imports `src/styles/global.css` (the only CSS import in the project)
- Loads Google Fonts: Sora + DM Sans
- Accepts props: `title` (required), `description` (optional), `canonical` (optional)
- Sets full SEO meta: description, robots, canonical, OG tags, Twitter card

### `Navbar.astro`
- Fixed position, `z-50`, `bg-white/95 backdrop-blur-sm`
- Adds `shadow-sm` on scroll via JavaScript
- Mobile menu: hamburger toggles hidden menu, animates bars to X
- CTA button "Get in Touch" links to `/contact`
- `aria-expanded` is toggled for accessibility

### `Hero.astro`
- Two-column layout on desktop: text left (44% width), dashboard right (flex-1)
- Background: `bg-linear-to-br from-slate-50 via-blue-50/30 to-white`
- Two decorative blur blobs (non-interactive, `pointer-events-none`)
- Two CTAs: "Explore Our Solution" (primary) + "Talk to an Expert" (outlined)
- Three stat items below CTAs with icon + title + description

### `DashboardMockup.astro`
- Purely visual SVG/HTML mockup of SAP Group Reporting dashboard
- Left sidebar: dark navy (`#0f1f3d`) with SAP branding and nav items
- Right panel: light gray (`#f8fafc`) with metric cards, bar chart, world map, donut charts
- Uses inline SVG for all charts — no chart libraries, no JavaScript
- Applied shadow: `shadow-dashboard`
- This component should never be interactive — it is decorative

### `WhyChoose.astro`
- Two-column: description left, 2×2 feature card grid right
- Four feature cards: Integrated & Flexible, Global Compliance, Smart Automation, Real-time Visibility
- Card hover: `hover:shadow-card-hover hover:border-primary-100`
- Icon background animates: `group-hover:bg-primary-100`

### `TrustedBy.astro`
- Full-width strip with "Trusted by Finance Leaders" label
- Five typographically-styled text logos: ALSCO, Dole, JABIL, LUXOTTICA, EVONIK
- Each uses a different font weight/style to simulate real logo variety
- No images — text only

### `CTA.astro`
- Dark gradient banner: `bg-linear-to-br from-navy-800 via-[#1a3a6e] to-[#0f2a5a]`
- Grid pattern overlay at 5% opacity (SVG `<pattern>`)
- Right side: decorative bar chart + donut chart SVGs at 30% opacity
- CTA button is white with `text-navy-900`

### `Footer.astro`
- Background: `bg-navy-900`
- 4-column grid: brand + description, empty (col-span-2), Company links, Contact info
- Company links mirror navbar exactly
- Contact info: email, phone, address with SVG icons
- Bottom bar: copyright year + Privacy Policy + Terms links

### `contact.astro` — Form
The contact form has four fields, all required:

| Field | Type | ID | Validation |
|---|---|---|---|
| Full Name | `text` | `name` | Non-empty string |
| Email Address | `email` | `email` | Valid email regex |
| Phone Number | `tel` | `phone` | Non-empty string |
| Query Details | `textarea` | `query` | Min 10 characters |

- Client-side validation only (this is a static site — no backend)
- On submit: shows spinner, simulates 1.4s async delay, shows green success banner
- Success banner auto-hides after 6 seconds
- Form resets after successful submission
- Error styling: `border-red-300` on invalid fields + hidden error `<p>` becomes visible

---

## 10. Design Reference & UI Rules

### Reference design
The site is modelled after the **FinCore** SaaS marketing page — a premium fintech enterprise UI. The color palette, layout proportions, component hierarchy, and dashboard mockup are derived from this reference.

### Brand identity
- **Logo:** Four vertical bars (ascending heights) in `primary-600`/`primary-400` + "Fin" in `text-navy-900` + "solves" in `text-primary-600`
- **Favicon:** Same bar chart concept, white bars on `primary-600` square with `rx-6` rounding

### Spacing conventions
- Section vertical padding: `py-16 lg:py-20` (standard) or `py-20 lg:py-24` (hero/major sections)
- Max content width: `max-w-7xl mx-auto`
- Horizontal padding: `px-4 sm:px-6 lg:px-8`
- Card internal padding: `p-6` (standard) or `p-7`–`p-8` (larger cards)

### Border radius conventions
- Buttons: `rounded-lg`
- Cards: `rounded-xl`
- Large panels / CTA blocks: `rounded-2xl`
- Icon containers: `rounded-lg` (small) or `rounded-xl` (large)

### Button styles
```
Primary:  bg-primary-600 text-white font-semibold rounded-lg hover:bg-primary-700 shadow-sm
Outlined: border border-slate-200 text-slate-700 hover:border-primary-300 hover:text-primary-600 hover:bg-primary-50 bg-white
White:    bg-white text-navy-900 font-semibold rounded-lg hover:bg-blue-50 shadow-sm
```

### Responsive breakpoints
- Mobile-first design
- `sm:` — 640px (two-column grids, side-by-side buttons)
- `md:` — 768px (navbar switches from hamburger to desktop)
- `lg:` — 1024px (two-column hero, section layout changes)
- `xl:` — 1280px (larger hero font sizes)

---

## 11. Security — CVEs Fixed & Version Constraints

### Patched vulnerabilities

| CVE / Advisory | Severity | Affected | Fixed in | Description |
|---|---|---|---|---|
| `GHSA-j687-52p2-xcff` | Moderate | `astro ≤6.1.9` | `astro 6.3.7` | XSS via incomplete `</script>` tag sanitization in `define:vars` |
| `GHSA-xr5h-phrj-8vxv` | Moderate | `astro ≤6.1.9` | `astro 6.3.7` | Server island encrypted parameters vulnerable to cross-component replay attack |

### Version floor rule

> **`astro` must never be downgraded below `6.3.7`.**
> Any version at or below `6.1.9` reintroduces both CVEs above.

### Upgrade history

| Date | Change | Reason |
|---|---|---|
| Initial build | `astro@^4.5.0` | Original version used |
| Upgrade 1 | `astro@^5.8.0` | Updated to latest at time of v2 |
| Upgrade 2 | `astro@^6.3.7` | Security patch for GHSA-j687 + GHSA-xr5h |

### Running audit

After any `npm install` or dependency change, always run:

```bash
npm audit
```

Expected output after install with these versions: **0 vulnerabilities**

If vulnerabilities appear, check `astro` version first — it must be `≥6.3.7`.

---

## 12. Dependency Rules

1. **No new dependencies without a documented reason.** This is a static site — it needs almost no runtime packages.
2. **No UI component libraries** (no shadcn, no Radix, no Headless UI). All components are hand-built with Tailwind.
3. **No CSS-in-JS libraries** (no styled-components, no Emotion).
4. **No chart libraries** (no Chart.js, no Recharts). All charts in `DashboardMockup.astro` are inline SVG.
5. **No animation libraries** for production builds. CSS transitions only (`transition-*` utilities).
6. **No icon libraries** (no Heroicons package, no Lucide). All icons are inline SVG paths.
7. If you must add a dependency, add it to `devDependencies` unless it is needed at runtime.

---

## 13. Configuration Files

### `astro.config.mjs`

```js
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  output: 'static',           // explicit static output — required in Astro 6
  site: 'https://finsolves.com',
  vite: {
    plugins: [tailwindcss()], // Tailwind v4 integration via Vite
  },
});
```

- `output: 'static'` is **required** in Astro 6 for static sites. Do not remove it.
- `site` must match the production domain for correct canonical URLs.
- Do not add `integrations: []` for Tailwind — that was the v3 approach. v4 uses `vite.plugins`.

### `tsconfig.json`

```json
{
  "extends": "astro/tsconfigs/strictest",
  "compilerOptions": {
    "strictNullChecks": true,
    "allowJs": true,
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler"
  }
}
```

- `astro/tsconfigs/strictest` is Astro 6's recommended preset (replaces `strict` from Astro 4/5)
- `moduleResolution: "bundler"` is required by Astro 6 — do not change to `node` or `node16`

### `src/styles/global.css`

```css
@import "tailwindcss";

@theme {
  /* Custom colors, fonts, and shadows only */
  /* Never add CSS rules or component styles here */
}
```

- This file must remain minimal — only `@import` and `@theme {}` tokens
- Imported once in `Layout.astro` — applies globally to all pages

---

## 14. Do's and Don'ts

### ✅ DO

- Use Tailwind utility classes for every single style
- Use `font-display` class for all headings and the logo
- Use `font-sans` (default) for body text
- Use `primary-600` as the default brand/action color
- Use `navy-900` for dark heading text
- Keep all SVG icons inline (copy the path, don't import a library)
- Run `npm audit` after any dependency change
- Keep `astro` at `^6.3.7` or higher
- Use `bg-linear-to-*` for gradients (Tailwind v4 syntax)
- Use `set:html` directive in Astro for rendering SVG path strings from JS arrays
- Keep components small and focused — one responsibility per component
- Update this `AGENTS.md` file when making structural changes

### ❌ DON'T

- Write CSS rules anywhere (no `<style>` tags, no `.css` class rules, no inline `style=""` attributes)
- Use `bg-gradient-to-*` (this is Tailwind v3 syntax — will silently fail in v4)
- Downgrade `astro` below `6.3.7`
- Add `@astrojs/tailwind` to the project (deprecated, replaced by `@tailwindcss/vite`)
- Create a `tailwind.config.mjs` or `tailwind.config.js` (not used in Tailwind v4)
- Add `tailwind.config.ts` (same — does not exist in v4)
- Import fonts anywhere other than `Layout.astro`
- Add new pages without updating the navbar
- Add new pages without registering them in this document
- Use `astro/tsconfigs/strict` (Astro 6 uses `strictest`)
- Add JavaScript that could be replaced with a CSS transition

---

## 15. Running the Project

### Install dependencies

```bash
cd finsolves
npm install
```

### Development server

```bash
npm run dev
# → http://localhost:4321
```

### Production build

```bash
npm run build
# → ./dist/ (static HTML/CSS/JS)
```

### Preview production build

```bash
npm run preview
# → http://localhost:4321
```

### Security audit

```bash
npm audit
# Expected: 0 vulnerabilities
```

### Minimum Node.js version

Astro 6 requires **Node.js 18.17.1 or higher**. Recommended: Node.js 20 LTS or 22 LTS.

```bash
node --version  # must be ≥18.17.1
```

---

## 16. Change History

### v1.0.0 — Initial build
- Created full Astro + Tailwind project from scratch
- Tech stack: Astro `^4.5.0`, Tailwind `^3.4.1`, `@astrojs/tailwind ^5.1.0`, TypeScript `^5.4.2`
- Pages: Home, Services, About Us, Contact
- Components: Navbar, Hero, DashboardMockup, WhyChoose, TrustedBy, CTA, Footer
- Design modelled after FinCore reference screenshot
- SAP Group Reporting dashboard mockup built with inline SVG
- Contact form with client-side validation (name, email, phone, query)
- SEO setup: meta description, OG tags, Twitter cards, canonical URLs

### v1.1.0 — Dependency upgrade (latest versions)
- Upgraded Astro: `^4.5.0` → `^5.8.0`
- Upgraded Tailwind: `^3.4.1` → `^4.1.0` (major version — breaking changes)
- Removed `@astrojs/tailwind` (deprecated in Tailwind v4)
- Added `@tailwindcss/vite ^4.1.0` (new Tailwind v4 integration method)
- Upgraded TypeScript: `^5.4.2` → `^5.7.0`
- Removed `tailwind.config.mjs` (no longer exists in Tailwind v4)
- Added `src/styles/global.css` (required Tailwind v4 CSS entrypoint)
- Moved all design tokens into `@theme {}` block in global.css
- Updated all gradient classes: `bg-gradient-to-*` → `bg-linear-to-*`
- Updated `astro.config.mjs`: replaced `integrations` with `vite.plugins`
- Updated `tsconfig.json`: added `moduleResolution: "bundler"`

### v1.2.0 — Security patch (CVE fixes)
- **SECURITY:** Upgraded Astro: `^5.8.0` → `^6.3.7`
- Patches `GHSA-j687-52p2-xcff`: XSS via `define:vars` script sanitization
- Patches `GHSA-xr5h-phrj-8vxv`: Server island cross-component replay attack
- Updated `tsconfig.json`: `astro/tsconfigs/strict` → `astro/tsconfigs/strictest` (Astro 6)
- Added `output: 'static'` to `astro.config.mjs` (required explicit in Astro 6)
