# AGENTS.md

Guide for AI agents working in this repository. Read this before touching anything.

This repo hosts **two independent products** that happen to deploy from the same
GitHub Pages site at `edsa.tech`. They have different stacks, different audiences
and different design languages. **Do not bleed one into the other.**

| # | Thing | Lives in | Stack | URL | Audience |
|---|-------|----------|-------|-----|----------|
| 1 | **EDSA — fintech/engineering portfolio + blog** | `src/` | Astro 5 (Bear Blog starter) | `https://edsa.tech/` | Engineers, recruiters, readers |
| 2 | **Gaby — iOS app marketing microsite** | `public/gaby/` | Hand-written static HTML + CSS, zero build step | `https://edsa.tech/gaby/` | Consumers / App Store visitors |

---

## Quick facts

- **Repo:** `github.com/delose/delose.github.io`, default branch `master`.
- **Custom domain:** `CNAME` → `edsa.tech`. Do not delete it.
- **Deploy:** push to `master` → `.github/workflows/deploy.yml` runs `withastro/action@v3`
  (build + upload) then `actions/deploy-pages@v4`. No manual step.
- **`astro.config.mjs` still sets `site: 'https://delose.github.io'`** even though the
  live domain is `edsa.tech`. That affects canonical URLs, the sitemap and RSS.
  Known wart — if you touch it, verify sitemap/RSS/canonicals across the whole site.
- **Analytics:** GA4 `G-4HC3PKE1DJ`, injected in `src/components/BaseHead.astro`.
  It is **only on Astro pages**. The Gaby pages are deliberately analytics-free.

### Commands

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # -> ./dist
npm run preview
```

There are no tests, no linter and no formatter configured. Don't invent a
`npm test` in docs or CI.

### Dead weight — ignore, don't "fix"

`_layouts/`, `_posts/`, `_config.yml`, `index.html` (Liquid), `css/`, `js/`, `json/`,
`svg/`, `workflows/` are leftovers from the old Jekyll site. `.nojekyll` disables
Jekyll, so none of it renders. `README.md` is still the unmodified Astro starter
README. Leave all of it alone unless explicitly asked to clean it up.

---

## 1. EDSA portfolio + blog (Astro)

```
src/
├── components/     BaseHead, Header, Footer, HeaderLink, FormattedDate
├── layouts/        BlogPost.astro
├── pages/          index.astro, about.astro, blog/index.astro, blog/[...slug].astro, rss.xml.js
├── content/
│   ├── blog/       microservices/*.mdx, ai-agents/*.mdx   <- published
│   └── archive/    not routed; starter leftovers
├── assets/         hero images imported by MDX frontmatter
├── styles/global.css
└── consts.ts       SITE_TITLE, SITE_DESCRIPTION
```

- **Content collections** are configured in `src/content.config.ts`. Blog posts are
  MDX with an image in frontmatter that must resolve to a file in `src/assets/`.
- **Styling:** a global Bear Blog stylesheet (`src/styles/global.css`) plus large
  `<style>` blocks scoped inside `.astro` pages. `src/pages/index.astro` carries its
  own ~400-line stylesheet. There is no Tailwind, no CSS framework, no design tokens
  file. Match the local convention of whatever file you're in.
- **Fonts:** self-hosted Atkinson Hyperlegible, preloaded in `BaseHead.astro`.
- **Adding a post:** create `src/content/blog/<category>/<slug>.mdx`, add the hero
  image to `src/assets/`, and it routes automatically via `blog/[...slug].astro`.
- **Known inconsistency:** `index.astro` links `/blog/fintech`, `/blog/microservices`
  and `/blog/ai-agents` as category pages and quotes article counts, but no category
  routes exist and the counts are hardcoded. Fix only if asked.

## 2. Gaby microsite (static HTML)

```
public/gaby/
├── gaby.css             the entire design system for all three pages
├── icon.svg             carnation mark — favicon + header logo
├── index.html           landing page
├── privacy/index.html   Privacy Policy  (App Store Connect "Privacy Policy URL")
└── support/index.html   Support + FAQ   (App Store Connect "Support URL")
```

**Design language.** Neo-brutalist, matching the app's own warmth: full-bleed saturated
colour blocks stacked in a fixed rhythm (cream hero → sun marquee → blue → cream → teal →
cream → terracotta → ink → coral → sun → ink footer), every block separated by a 3 px black
rule; cream cards with `3px solid ink` borders and a hard `6px 6px 0` offset shadow; huge
tight headlines; monospace uppercase micro-labels (`.mono`, `.eyebrow`) for kickers, meta
and footer column heads. Palette tokens live at the top of `gaby.css`
(`--terracotta #c1440e`, `--coral`, `--teal`, `--blue`, `--sun`, `--ink`, `--paper`).
The hero iPhone is **CSS, not an image** (`.phone` / `.ui-*`) — it reproduces the app's
"At the tapas bar" correction screen. Update it if that screen changes.

**Why it isn't Astro.** It lives in `public/` on purpose. Astro copies `public/`
verbatim to `dist/`, so these pages ship byte-for-byte with no build step, no
framework, no JS and no analytics — which is the point: Gaby's whole pitch is that
it collects nothing and phones home never, and the marketing site should match. It
also keeps the product's brand fully separate from the blog chrome (`Header.astro`
/ `Footer.astro` must **never** appear on a Gaby page).

**Editing rules**

- All three pages share `/gaby/gaby.css`. Put changes there, not in per-page
  `<style>` blocks, so the pages stay consistent.
- Keep it dependency-free: **no** Google Fonts, no CDN scripts, no analytics, no
  tracking pixels, no external images, **no JavaScript** (the FAQ accordions are native
  `<details>`). System font stack only — a page that brags about zero network requests
  should not make any. If a design ever needs a webfont, self-host it in `public/gaby/`.
- Hand-placed `<br>` in headings must have **a space before it** (`Spanish <br>you'll`),
  because `gaby.css` hides those breaks below 680 px and the space is what keeps the words
  apart when it does.
- The privacy and support URLs are registered in App Store Connect. **Never rename
  or remove `/gaby/privacy/` or `/gaby/support/`** — a 404 there can hold up review.
- Links are absolute from the site root (`/gaby/privacy/`), not relative.

**Product facts — keep these accurate and identical across all three pages,
the App Store listing and the app itself:**

- Name on the App Store: **Gaby: Speak Spanish**. Subtitle: *Rehearse it before you live it*.
- App Store ID `6790808283` → `https://apps.apple.com/app/id6790808283`
- **$14.99 one-time**, iPhone, no subscription, no in-app purchases, 4+, Education.
- 24 situations · 3 levels each (Guided / Practice / Real) · 100 lessons ·
  124-piece collection of Spanish paintings and places · UI in 12 languages.
- Fully offline: **makes no network requests at all.** No account, no ads, no streaks.
- Speech is transcribed **on device** (`requiresOnDeviceRecognition`); voice is never
  recorded to a file or transmitted.
- Every Spanish sentence and every correction is **written by a person**, fixed at
  release. Nothing is generated. Earlier versions downloaded a ~1.8 GB Hugging Face
  model on first launch — **that is gone**, and several pages say so explicitly.
- Support email: `support@edsa.tech`. Developer: Eugene Sergio Castro De Los Santos.

**Do not** claim ratings, download counts, testimonials, awards, or a Mac/iPad/Android
version. Do not add a subscription, trial, newsletter or waitlist. If a number in the
copy needs to change, change it in *every* place listed above.

---

## Conventions

- Tabs for indentation in `.astro` files; 2 spaces in the Gaby HTML/CSS. Follow the file.
- Absolute root-relative links (`/blog`, `/gaby/support/`), trailing slash on directory URLs.
- Commit style is short and imperative, e.g. `Update Gaby pages for v2.0`.
- Only commit or push when the user asks.

## Before you finish

1. `npm run build` must succeed.
2. If you touched `public/gaby/`, open every one of the three pages and check them at
   ~390 px and ~1440 px wide. Verify the App Store link, the `mailto:` and the
   privacy/support cross-links.
   Note: `astro dev` serves these as `/gaby/index.html`, not `/gaby/` — only the built
   site and GitHub Pages resolve the directory URL. `.claude/launch.json` starts the dev
   server on :4321. Headless Chrome is the reliable way to screenshot them:
   `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --screenshot=out.png --window-size=1440,900 <url>`
   (to check a narrow viewport, load the page in a fixed-width `<iframe>` — `--window-size`
   alone does not force a narrow layout).
3. If you changed a product fact, grep for it across `public/gaby/` and fix all copies.
