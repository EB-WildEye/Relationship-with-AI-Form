# My Toxic Relationship With AI — Take Away

A single-page Hebrew (RTL) take-away for the *My Toxic Relationship With AI*
workshop. Attendees scan a QR code at the end of the talk and land here, so the
page is built phone-first.

## How it works

The page presents three pattern cards — recurring habits in how people talk to
AI models, each with a chat example from the talk, a copy-able example prompt, a
short exercise, and a textarea for the attendee's own answer.

At the bottom they add their name and field of work, optionally ask for personal
feedback, and press one button. That button assembles everything they wrote into
a message and opens WhatsApp through a `wa.me` deep link, pre-filled and ready to
send. Empty answers are skipped; if nothing was filled in, a short fallback line
is sent instead.

## No backend

There is no server, no database, and no analytics. Nothing typed into the page is
stored or transmitted anywhere — the message is assembled in the browser and
handed to WhatsApp, so the answers only ever exist in the attendee's own message,
sent from their own account.

## Tech

One `index.html` with its CSS and JavaScript inline, plus self-hosted WOFF2 fonts
and a preview image. No framework, no dependencies, no build step. Deployed as a
static site on Vercel.

Fonts are self-hosted rather than loaded from Google Fonts, both to avoid a
third-party request on every visit and because the Hebrew handwriting face
carries the design and must not fall back to a system cursive.

## Running locally

Any static file server works. The page must be served over HTTP rather than
opened from the filesystem, so the fonts resolve:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

## Deploying

Import the repository at [vercel.com/new](https://vercel.com/new), set the
Framework Preset to **Other**, and leave the Build Command and Output Directory
empty. There is nothing to build. Every push to `main` redeploys.

The Open Graph tags in `<head>` use absolute URLs, because link previews will not
resolve relative paths. They sit together in one marked block and must match the
domain the site is actually served from.

## Implementation notes

Three things here look arbitrary and are not. Changing them will break the page
in ways that are easy to miss.

**The 400-character limit on each answer textarea.** Hebrew expands
substantially once percent-encoded into a URL, and the assembled `wa.me` link has
to stay within a length every mobile browser will actually navigate to. Raising
the limit means re-measuring the worst-case encoded URL first.

**The WhatsApp click handler is fully synchronous.** No `await`, no promise, and
no timer runs before the navigation. iOS Safari only honours `window.open` and
location changes inside the original user gesture, so introducing any
asynchronous step before the hand-off silently breaks the button on iPhone.

**The copy buttons use `document.execCommand("copy")`** with a select-the-text
fallback, rather than the modern Clipboard API. That API is unavailable in
insecure contexts and inside several in-app browsers — which is exactly where a
page opened from a QR code tends to be viewed.
