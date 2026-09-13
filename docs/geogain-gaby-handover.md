# Handover: the Gaby quiz answer pages

Paste this into the Claude session that works on the **GeoGain** repo
(`/Users/eugenedelossantos/prototype/GeoGain`). Written to be read cold.

---

## What changed, and why it matters to you

`edsa.tech/gaby/quizzes/<slug>/` now serves **100 answer pages generated directly from
GeoGain's own database** — `content_queue`, `niche='spanish'`, the `quiz` object inside
`script_json`. They are finally in sync with what GeoGain posts.

Before this, the site had 100 unrelated quizzes written by hand in the site repo. Those
are gone.

## The contract between the two repos

The **site repo** (`/Users/eugenedelossantos/prototype/delose.github.io`) owns:

```bash
npm run quiz:import   # reads GeoGain's db/geogain.db -> tools/quizzes.json (committed snapshot)
npm run quiz:build    # -> public/gaby/quizzes/<slug>/ + tools/quiz-manifest.json
npm run build && git push    # deploys via GitHub Actions
```

**GeoGain owns the quiz content, the card images and the captions.** The site repo
never invents a quiz. If a quiz changes in GeoGain, re-run the two commands above in
the site repo and push.

## What GeoGain needs to add

Under the existing **Quiz card** section in the admin (next to "Copy answer" and the
4:5 / 9:16 exports), add the answer-page link so it can be **copied, clicked, and
opened in a new tab**:

```html
<div class="answer-link">
  <span class="label">Answer page — goes in comment 2</span>
  <a href="https://edsa.tech/gaby/quizzes/{slug}/" target="_blank" rel="noopener">
    edsa.tech/gaby/quizzes/{slug}
  </a>
  <button onclick="navigator.clipboard.writeText('https://edsa.tech/gaby/quizzes/{slug}/')">
    Copy link
  </button>
</div>
```

## Where `{slug}` comes from

Read **`tools/quiz-manifest.json`** in the site repo. It is regenerated on every build
and is deliberately **not deployed**, because it contains every answer. Each entry
carries `geogainId`, which is `content_queue.id` — join on that, no matching by text:

```json
{
      "order": 1,
      "geogainId": "0070df5e-66b1-4c0f-a89e-b30031b8d8b7",
      "slug": "say-hello-in-spanish",
      "url": "https://edsa.tech/gaby/quizzes/say-hello-in-spanish/",
      "topic": "Say Hello in Spanish",
      "question": "How do you say 'Good evening' or 'Good night' in Spanish, say at 8 PM?",
      "options": [
            "Buenas tardes",
            "Buenas noches",
            "Buenos días"
      ],
      "answerIndex": 1,
      "answer": "Buenas noches",
      "why": "Buenas noches is for evening and night."
}
```

So in the admin, for the row you are showing, look up the manifest entry whose
`geogainId` equals that row's `id` and use its `url`.

## Rules that will bite you

1. **A slug is a permanent public URL** once the comment is posted. It is derived from
   the lesson title. **Changing a lesson title changes the slug and breaks every
   comment already posted for it.** If a title must change after posting, tell the site
   repo to pin the old slug via `SLUG_OVERRIDES` in `tools/import-from-geogain.mjs`.
2. **Slugs are ASCII only.** `conversación` would percent-encode to `conversaci%C3%B3n`,
   which looks broken pasted into a comment.
3. **A slug must never give the answer away.** The site build fails if it does. One
   already needed an override: the lesson "Me gustaría" has "Me gustaría la cuenta" as
   its answer, so it is published as `pedir-con-cortesia`.
4. **Two comments, never one.** Comment 1 is the quiz card image with no answer on it.
   Comment 2, posted later, is the answer-page link. Putting the link in the post kills
   the comment engagement the quiz exists to create.
5. **No day numbers on the public pages, on purpose.** A page saying "Day 16" cannot be
   re-shared. Ordering stays inside GeoGain.

## What an answer page contains

Question and options → the answer behind a "Tap to reveal" `<details>` → the
explanation → a promotional YouTube Short → the App Store CTA.

The Short is **click-to-load**: nothing is requested from Google until the visitor
presses play. If you change which video it is, it is the `YT` constant in
`tools/build-quiz-pages.mjs`, and `/gaby/privacy/` section 11 describes the embed.

## Measuring it

Gaby pages carry **GoatCounter** (<https://edsa.goatcounter.com>, cookieless, no IP
stored). Every answer page is tagged, so `/gaby/quizzes/<slug>/` in the **Pages** list
is a direct count of clickthroughs from a comment, and **Referrers** shows which
platform sent them. That is the number that says which quizzes are worth repeating.
