# My Toxic Relationship With AI — Take Away

The one-page Hebrew (RTL) take-away for the *My Toxic Relationship With AI*
workshop. Attendees scan a QR code at the end of the talk and land here.

The page holds three pattern cards — each with a chat example from the talk, a
copy-able example prompt, a numbered exercise, and a textarea for the attendee's
own answer. At the bottom they enter their name and business, optionally tick a
box asking for personal feedback, and press one button that opens WhatsApp with
everything pre-filled as a message.

**There is no backend and there must never be one.** Nothing is stored, nothing
is posted anywhere. The button assembles a `wa.me` deep link in the browser and
hands off to WhatsApp. Answers exist only in the attendee's own message.

---

## The two things you will actually want to change

### 1. The WhatsApp number

One constant, at the top of the `<script>` block near the bottom of
`index.html`:

```js
const PHONE = "972532348301";
```

International format, **digits only** — no `+`, no leading zero, no dashes.
`972` is Israel, followed by the number without its leading `0`
(`053-234-8301` → `53234-8301` → `972532348301`).

Change it in that one place and everything follows: the deep link, the
`053-234-8301` shown in the failure fallback panel, and the retry link. Nothing
else in the file contains the number.

### 2. The deployed domain

Link previews need absolute URLs — WhatsApp will not resolve a relative
`og:image`. The domain appears in exactly one marked block in `<head>`:

```html
<!-- ==== ABSOLUTE URLS - THE ONLY PLACE THE DOMAIN APPEARS ==== -->
<meta property="og:url"     content="https://relationship-with-ai.vercel.app/">
<meta property="og:image"   content="https://relationship-with-ai.vercel.app/og-image.png">
<meta name="twitter:image"  content="https://relationship-with-ai.vercel.app/og-image.png">
```

Those three lines are the only occurrences of the domain anywhere in the
project — verified, not assumed. If it changes — a custom domain, or Vercel
suffixing the name because `Relationship-with-AI` was taken — edit them and
nothing else.

After changing it, re-share the link through
[Facebook's Sharing Debugger](https://developers.facebook.com/tools/debug/) to
flush WhatsApp's preview cache, or WhatsApp will keep showing the old preview
for days.

---

## Deploying

Pure static site. **No build step, no dependencies, no `package.json`.** Vercel
serves the repo root as-is.

### First deploy — via GitHub (recommended)

1. Create the repo and push:
   ```bash
   git remote add origin https://github.com/<you>/relationship-with-ai.git
   git push -u origin main
   ```
2. On [vercel.com/new](https://vercel.com/new), import the repo.
3. **Set the Project Name to `Relationship-with-AI`** — this is what produces
   `relationship-with-ai.vercel.app`, and it is independent of the folder or repo name.
4. Framework Preset: **Other**. Leave Build Command and Output Directory empty.
5. Deploy.

### First deploy — via CLI

```bash
npx vercel          # first run prompts for the project name — enter Relationship-with-AI
npx vercel --prod   # promote to production
```

(The old `--name` flag was removed from the Vercel CLI; the name comes from the
first-run prompt, and is editable afterwards under **Project Settings → General
→ Project Name**.)

### Redeploying

Every push to `main` redeploys production automatically. Pushes to any other
branch get their own preview URL. To redeploy without a code change, use
**Deployments → ⋯ → Redeploy** in the Vercel dashboard.

---

## What's in here

```
index.html        the entire page — HTML, CSS and JS inline, single file
og-image.png      1200×630 link preview card, 109 KB
fonts/            six self-hosted woff2 subsets
vercel.json       clean URLs + cache headers, no build config
```

### Fonts

Self-hosted, **no requests to fonts.googleapis.com**. The Hebrew handwriting
face (Gveret Levin) carries the design, so it must never silently fall back to a
system cursive.

| File | Family | Covers |
|---|---|---|
| `rubik-var-{hebrew,latin}.woff2` | Rubik | weights 300–600 via variable axis |
| `gveret-levin-{hebrew,latin}.woff2` | Gveret Levin | 400 |
| `instrument-serif-{normal,italic}-latin.woff2` | Instrument Serif | 400 |

Rubik is a variable font, so one file per subset covers all four weights the
page uses — that is why there are six files and not twelve. Each `@font-face`
keeps its original `unicode-range`, so a phone still downloads only the Hebrew
subset until it hits a Latin glyph. Total ~128 KB, cached for a year.

To update a font, re-download the woff2 from Google Fonts and keep both the
filename and the `unicode-range` in `index.html` unchanged.

---

## Things not to break

**The 400-character limit** on each of the three answer textareas is load-bearing,
not cosmetic. A Hebrew letter is two UTF-8 bytes, and each byte becomes a
three-character percent triplet, so Hebrew costs **6 URL characters per typed
character** (`ל` → `%D7%9C`).

Measured in Chrome with all three textareas at their full 400 Hebrew characters,
name and business at their full 60, and the feedback box ticked:

| | chars |
|---|---|
| 3 × 400 Hebrew chars | 7,200 |
| name + business (60 each, capped) | 720 |
| labels, headers, newlines, ASCII | ~585 |
| base `https://wa.me/…?text=` | 31 |
| **measured total** | **8,536** |

That sits well inside the ~32,000-character practical navigation limit on iOS
Safari, Android Chrome and desktop. Projections if the limit ever moves:

| textarea limit | worst-case URL |
|---|---|
| 400 (today) | 8,536 |
| 600 | ~12,100 |
| 1,000 | ~19,300 |
| 1,500 | ~28,300 — close to the edge, do not go here |

**Re-measure before changing the limit** rather than trusting these projections:
emoji are four UTF-8 bytes and cost **12** URL characters each, so a field full
of emoji is twice as expensive as the same field full of Hebrew. The name and
business inputs are capped at 60 characters each for the same reason.

**The copy buttons** use `select` → `document.execCommand("copy")` with a
`"מסומן - Ctrl+C"` fallback state, deliberately — the async Clipboard API is
unavailable in insecure contexts and inside some in-app browsers, which is
exactly where workshop attendees open this page from. Do not "modernise" them.

**The WhatsApp click handler is fully synchronous.** No `await`, no promise, no
timer runs before the navigation. iOS Safari only honours `window.open` and
`location` changes inside the original user gesture; introducing any async step
before the hand-off silently breaks the button on iPhone. On mobile it navigates
with `location.href`; on desktop it tries `window.open` and falls back to
`location.href` in the same tick if a popup blocker returns `null`. If no
navigation has happened 1.4 s later, a Hebrew fallback panel appears with the
full message in a selectable textarea and a copy button.

---

## Narrow phones

Almost every visitor arrives on a phone from a QR code, so there is one media
query at `max-width: 379px`. It does not fire on any mainstream phone in
portrait (iPhone SE is 375px CSS wide, but so is the iPhone 13 mini — check
before assuming); it exists for the 320–360px long tail and for landscape
splits.

Inside it: page gutter 20px → 14px, card padding 26px → 18px, send-block padding
28px → 20px, path-pill padding 15px → 12px, and `.prompt` reserves 70px instead
of 56px on its inline-end.

That last one is a real bug fix, not a tweak. The copy button reaches **67px**
in from the prompt's inline-start edge, but the original reserved only 56px, so
on a 320px screen the button sat visibly on top of the first line of Hebrew.
The same 11px overlap still exists above 380px; it just is not visible there,
because the lines happen to wrap before reaching the button. If you ever change
the copy button's label, font size or padding, re-measure its width and keep
`padding-inline-end` above it.

Verified at 320, 360, 375, 379, 380, 414, 768 and 1280px: **zero horizontal
overflow at every width.**

---

## Accessibility

Labels are associated, the textareas carry `aria-label` plus `aria-describedby`
pointing at their character counters, decorative `✱` marks are `aria-hidden`,
Latin runs inside the Hebrew document are marked `lang="en"` so screen readers
switch voice, and keyboard focus is visible via `:focus-visible`. All body text
meets WCAG AA contrast.

Form fields are 16px because anything smaller makes iOS Safari zoom the page on
focus.
