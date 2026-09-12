# Astro Starter Kit: Blog

```sh
npm create astro@latest -- --template blog
```

> 🧑‍🚀 **Seasoned astronaut?** Delete this file. Have fun!

Features:

- ✅ Minimal styling (make it your own!)
- ✅ 100/100 Lighthouse performance
- ✅ SEO-friendly with canonical URLs and OpenGraph data
- ✅ Sitemap support
- ✅ RSS Feed support
- ✅ Markdown & MDX support

## 🚀 Project Structure

Inside of your Astro project, you'll see the following folders and files:

```text
├── public/
├── src/
│   ├── components/
│   ├── content/
│   ├── layouts/
│   └── pages/
├── astro.config.mjs
├── README.md
├── package.json
└── tsconfig.json
```

Astro looks for `.astro` or `.md` files in the `src/pages/` directory. Each page is exposed as a route based on its file name.

There's nothing special about `src/components/`, but that's where we like to put any Astro/React/Vue/Svelte/Preact components.

The `src/content/` directory contains "collections" of related Markdown and MDX documents. Use `getCollection()` to retrieve posts from `src/content/blog/`, and type-check your frontmatter using an optional schema. See [Astro's Content Collections docs](https://docs.astro.build/en/guides/content-collections/) to learn more.

Any static assets, like images, can be placed in the `public/` directory.

## 🧞 Commands

All commands are run from the root of the project, from a terminal:

| Command                   | Action                                           |
| :------------------------ | :----------------------------------------------- |
| `npm install`             | Installs dependencies                            |
| `npm run dev`             | Starts local dev server at `localhost:4321`      |
| `npm run build`           | Build your production site to `./dist/`          |
| `npm run preview`         | Preview your build locally, before deploying     |
| `npm run astro ...`       | Run CLI commands like `astro add`, `astro check` |
| `npm run astro -- --help` | Get help using the Astro CLI                     |

## 👀 Want to learn more?

Check out [our documentation](https://docs.astro.build) or jump into our [Discord server](https://astro.build/chat).

## Credit

This theme is based off of the lovely [Bear Blog](https://github.com/HermanMartinus/bearblog/).

## Analytics — the two dashboards

Two tools, split by which part of the site they cover. Both are owner-only.

| Dashboard | Covers | Link |
|---|---|---|
| **GoatCounter** | `edsa.tech/gaby/**` — the app landing page, privacy, support, and every quiz answer page | <https://edsa.goatcounter.com> |
| **GA4** | Everything Astro renders — `/`, `/blog`, `/about`, blog posts | [Google Analytics property](https://analytics.google.com/analytics/web/#/a382128010p521752320/reports/intelligenthome) |
| GitHub traffic | Repo-level views and referrers, 14-day window | [Insights → Traffic](https://github.com/delose/delose.github.io/graphs/traffic) |

They do not overlap: GA4 is injected by `src/components/BaseHead.astro`, which only
runs on Astro pages; the Gaby pages are hand-written HTML and carry the GoatCounter
tag instead. So no page is double-counted.

### Reading GoatCounter

Log in at <https://edsa.goatcounter.com> (site code `edsa`). The front page *is* the
dashboard — no navigating. **Pages** lists each URL with its visitor count, so
`/gaby/quizzes/tapas-bar/` tells you how many people actually clicked through from a
social comment; **Referrers** shows which platform sent them. Change the period with
the date control at the top right.

### Reading GA4

1. Go to the [property link above](https://analytics.google.com/analytics/web/#/a382128010p521752320/reports/intelligenthome)
   and sign in as the account that owns property `G-4HC3PKE1DJ`.
2. Left sidebar → **Reports**.
3. **Reports snapshot** for the overview; **Life cycle → Engagement → Pages and
   screens** for per-page numbers; **Life cycle → Acquisition → Traffic acquisition**
   for where visitors came from.
4. Set the date range top right — it defaults to the last 28 days.
5. **Realtime** (under Reports) shows the last 30 minutes, which is the quickest way
   to confirm tracking is alive after a deploy.

GA4 processing lags by up to 24–48 hours; Realtime does not. GoatCounter is near
instant.