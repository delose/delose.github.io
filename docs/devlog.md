# The devlog, and how `/log/` gets its entries

One append-only log records what actually shipped, across every project and every
Claude session on this machine. Three things read it:

1. **Other sessions** — a session in one repo can see what happened in another.
2. **X post drafts** — the `tweets` skill writes from it and may not invent a fact.
3. **This site** — `/log/` renders the public subset.

```
~/.claude/devlog/
├── events.jsonl     the log itself, one JSON object per line, append-only
├── bin/devlog       the CLI
├── profile.md       posting voice and the fixed facts, for the tweet drafts
└── projects.json    the project registry (nope, gaby, bubbled, geogain, edsa.tech, other)

~/.claude/CLAUDE.md          loaded into every Claude Code session on this machine
~/.claude/skills/devlog/     the full logging rules
~/.claude/skills/tweets/     drafts X posts from the log
```

**The log lives outside this repo on purpose.** It spans every project and it holds
entries that are not public. Only the `public: true` subset reaches the site.

## What this repo holds

| Path | What |
|---|---|
| `src/data/changelog.json` | **Generated. Never hand-edit.** Written by `devlog publish`. |
| `src/pages/log/index.astro` | Renders it, grouped by month, newest first. |

`src/components/Header.astro` carries the `Log` nav item. Four items need tighter
padding than three below 720 px, which is why that media query sets `padding: 1em
0.25em` — without it the nav wraps to three lines on a phone.

An entry's `id` is its anchor, so `edsa.tech/log/#2026-11-01-nope-nope-approved-on-the-app-store`
links straight to one entry.

## The loop

```bash
# 1. a session logs a milestone (usually automatic, see ~/.claude/CLAUDE.md)
~/.claude/devlog/bin/devlog add --project nope --kind approved \
  --title "..." --detail "..." --angle "..." --public --tweetable high

# 2. draft posts from it — ask any session for "tweets", or run the tweets skill
~/.claude/devlog/bin/devlog pending

# 3. after posting, record the URL. This is what keeps X and the site in sync.
~/.claude/devlog/bin/devlog posted <id> https://x.com/eugenesergio/status/...

# 4. export and ship
~/.claude/devlog/bin/devlog publish --repo "$PWD"
npm run build && git add -A && git commit -m "Log: ..." && git push
```

Step 3 is the part that matters for "my X and my logs are synced": the entry on
`/log/` then carries an **on X →** link to the post that announced it, and the
generator stops offering that event again.

An entry can be public without ever being tweeted, and tweeted without being public.
`public` and `tweetable` are separate fields.

## Re-publishing

`devlog publish` rewrites `src/data/changelog.json` from scratch every time, so it also
picks up edits and newly-flipped `--public` flags. Run it before any commit that is
meant to change `/log/`.

---

## The prompt to paste into sessions that don't load `~/.claude/CLAUDE.md`

Claude Code on this Mac picks the rules up automatically — **nothing to paste**. Paste
this only in a session that cannot read that file: claude.ai in a browser, a phone, a
different machine, or another assistant.

```text
Keep my central devlog updated.

I keep one append-only log of what I actually ship, at
~/.claude/devlog/events.jsonl on my Mac. It feeds my X posts (@eugenesergio) and
the public changelog at edsa.tech/log/, so it has to be accurate and thin.

At the end of this session, if we shipped something real — it went live, it was
submitted to a store, it was approved or rejected, a feature a user would notice is
done, a real number came in, or something was cut for a reason worth remembering —
record ONE entry.

Most sessions record nothing. Refactors, typos, docs, explorations and work in
progress are not milestones, and padding the log is worse than leaving it empty.
Never put an API key, token, password or personal data in it; part of it is
published to a website.

If you have a shell on my Mac, run:

  ~/.claude/devlog/bin/devlog add \
    --project <nope|gaby|bubbled|geogain|edsa.tech|other> \
    --kind <shipped|submitted|approved|rejected|built|fixed|learned|decided|killed|metric|milestone> \
    --title "one plain line, past tense" \
    --detail "one to three sentences: what changed and why it mattered" \
    --angle "why someone who isn't me would care — the contrast or the surprise" \
    --numbers "a real figure|another" \
    --tweetable <high|medium|low|no> \
    [--public]   # only if it is ALREADY visible to the world

If you have no shell, print the single JSON line instead and tell me to run
`devlog add --json '<line>'` myself. Fields: id, ts, project, kind, title, detail,
numbers[], angle, links[], tags[], public, tweetable, posted, source{}.

The `angle` field is the valuable one. A diff is not an angle. Leave it empty
rather than writing filler.

Then tell me in one line what you logged.
```
