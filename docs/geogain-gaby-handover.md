# Handover: wire the Gaby quiz answer pages into the GeoGain posting pipeline

Paste this whole file into the Claude session that works on the **GeoGain** repo.
It is written to be read cold — it assumes you know nothing about the other repo.

---

## What already exists (do not rebuild any of it)

`edsa.tech` is a GitHub Pages site built from `github.com/delose/delose.github.io`.
Inside it sits **Gaby**, an iOS app marketing microsite at `edsa.tech/gaby/`.

As of this handover, **100 public answer pages are live**, one per quiz:

```
https://edsa.tech/gaby/quizzes/<slug>/

e.g.  https://edsa.tech/gaby/quizzes/tapas-bar/
      https://edsa.tech/gaby/quizzes/palabras-en-cion/
      https://edsa.tech/gaby/quizzes/ser-o-estar/
```

Each page: restates the question and its three options, hides the answer behind a
native `<details>` ("Tap to reveal"), gives the reason and a one-line takeaway,
then pitches the app with an App Store button. No JavaScript, no day number.

**They are already built and deployed. GeoGain does not generate them.** GeoGain's
job is only to *reference* them.

## The one thing GeoGain needs to add

Under the existing **Quiz card** section (the one with "Copy answer", the 4:5 and
9:16 exports, and the "Reply to paste once it has run" box), add a row for the
answer-page link that can be **copied, clicked, and opened in a new tab**:

```html
<div class="answer-link">
  <span class="label">Answer page — goes in comment 2</span>
  <a href="https://edsa.tech/gaby/quizzes/{{slug}}/" target="_blank" rel="noopener">
    edsa.tech/gaby/quizzes/{{slug}}
  </a>
  <button onclick="navigator.clipboard.writeText('https://edsa.tech/gaby/quizzes/{{slug}}/')">
    Copy link
  </button>
</div>
```

It belongs next to "Copy answer", because it is the other half of the same step.

## Where `{{slug}}` comes from

The source of truth lives in the other repo at `tools/quizzes.json`, and
`npm run quiz:build` emits a machine-readable manifest at
**`tools/quiz-manifest.json`** (deliberately NOT deployed — it contains answers):

```json
{
  "generated": "2026-09-12",
  "base": "https://edsa.tech/gaby/quizzes/",
  "count": 100,
  "quizzes": [
    {
      "order": 1,
      "slug": "tapas-bar",
      "url": "https://edsa.tech/gaby/quizzes/tapas-bar/",
      "topic": "At the tapas bar",
      "place": "a neighbourhood bar in Madrid",
      "question": "You are at the bar and want to ask what tapas they have. Which one is right?",
      "options": ["¿Qué tapas tiene?", "¿Dónde están las tapas?", "¿Qué tapas tienen?"],
      "answer": "¿Qué tapas tienen?",
      "takeaway": "Addressing a business is almost always plural in Spain."
    }
  ]
}
```

Pick one:

- **Copy it in.** Simplest. Drop the file into GeoGain and match on `slug`, or on
  `order` if GeoGain's own day numbers line up.
- **Match on your own content.** If GeoGain already owns its quiz text, add a
  `gabySlug` field to each GeoGain quiz and paste the slug in by hand. The manifest
  is then just a lookup list.

Ask the repo owner which, because it depends on whether GeoGain's quizzes are the
same 100 as the site's. **They may not be** — the site's 100 were written
independently and cover the app's own situations plus Spanish-for-Spain grammar.

## Rules that will bite you if you ignore them

1. **A slug is a permanent public URL.** It goes into social comments that outlive
   any redesign. Never change one after the post goes out, and never reuse one.
2. **Slugs are ASCII only** — `[a-z0-9-]`. No accents. `conversación` in a URL
   percent-encodes to `conversaci%C3%B3n`, which looks broken when pasted into a
   comment and defeats the point of a readable link.
3. **A slug must never give the answer away.** It names the *choice*, not the
   winner: `ser-o-estar`, `aparcar-o-estacionar`, `que-o-cual` — never
   `para-llevar` when "para llevar" is the correct option. The site's build fails
   on this automatically; GeoGain should not invent slugs that bypass it.
4. **There is no day number on the public pages, on purpose.** A quiz that says
   "Day 16" cannot be re-shared later. Keep day/ordering entirely inside GeoGain.
5. **Two comments, never one.** Comment 1 is the quiz card image with no answer on
   it. Comment 2, posted later, is the answer-page link. Putting the link in the
   post itself kills the comment engagement the quiz exists to create.

## Adding a 101st quiz

It is a change in the **site** repo, not GeoGain:

1. Append an object to `tools/quizzes.json` with `day`, `slug`, `situation`,
   `situationEn`, `place`, `question`, `options[]` (3), `answer`, `why` (may
   contain `<b>`/`<i>`), `takeaway`.
2. `npm run quiz:build` — validates required fields, slug format, duplicate slugs,
   duplicate questions and answer-revealing slugs, then writes the page and
   refreshes the manifest.
3. `npm run build`, commit, push to `master`. GitHub Actions deploys it.

## Analytics, so you can tell whether any of this works

Gaby pages carry **GoatCounter** (`https://edsa.goatcounter.com`, cookieless, no
IP stored). Every answer page is tagged, so
`/gaby/quizzes/tapas-bar/` in the **Pages** list is a direct count of how many
people clicked through from a comment, and **Referrers** shows which platform sent
them. That is the number that tells you which quizzes are worth repeating.

The Astro pages (`/`, `/blog`) use GA4 instead — different tool, no overlap.

## One caveat worth repeating to the owner

Gaby's core marketing claim is that its Spanish is **written by a person and never
generated**. The 100 quizzes on the site were drafted by an AI assistant and are
explicitly pending human review. A generated Spanish error in a public comment
would undercut the exact claim the app sells on, so nothing should be posted before
the owner has read it.
