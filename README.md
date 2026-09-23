# AT13 Hair Design — 基隆仁三店

Static site for AT13 Hair Design, served from a Cloudflare Worker.

Live: <https://at13-hair-design.wahmey.workers.dev/>

## ⚠️ This is a snapshot of the *built* site, not its source

The files in `public/` were mirrored from the live deployment on 2026-08-11.
They are Vite build output: the JavaScript is minified and bundled into
content-hashed chunks, and no source maps were published, so the original
`src/` tree **cannot be recovered from this snapshot**.

The bundles still name the source modules they came from (`src/ui/preloader`,
`src/ui/cursor`, `src/ui/nav`, `ui/sections/render/footer.js`), which confirms
the original project exists somewhere — it was just never on this machine. If
you still have it, commit it alongside `public/` and treat this snapshot as a
fallback.

What this snapshot **is** good for: it deploys, it renders identically to
production, and it puts the artifact under version control instead of leaving
it only on Cloudflare.

What it is **not** good for: editing. Changing copy or layout means editing
minified bundles by hand.

## Contents

| Path | What it is |
| --- | --- |
| `public/index.html` | Page shell, metadata, `HairSalon` JSON-LD |
| `public/assets/*.js` | 18 bundled chunks — app entry, three.js, GSAP, content, i18n dictionary |
| `public/assets/*.css` | 5 stylesheets |
| `public/assets/*.webp` | 95 images (full, texture and thumbnail variants) |
| `public/assets/gen/**/*.webp` | 10 real designer portraits — see below |
| `source/designer-cards/` | The owner's original designer cards, and how to re-crop them |
| `tools/crop-designer-cards.py` | Card → portrait, all three sizes |
| `public/assets/*.woff2` | Subset Noto Serif TC |
| `public/robots.txt` | Blanket `Disallow: /` — see below |
| `wrangler.jsonc` | Worker config, static assets only |

The site is bilingual (zh-Hant-TW / en) and uses a WebGL canvas driven by
three.js, with GSAP for transitions.

## The designer photos

The team section shows the salon's own photography. The owner supplied ten
finished designer cards (portrait plus name, number and 擅長項目, laid out in
their own house style); the portrait was cropped out of each card to 3:4 and
the wording was moved into the site's bilingual content data, so the text
stays selectable, translatable and responsive instead of being baked into a
picture.

The cards themselves are kept in `source/designer-cards/` — they are the
masters, and the portraits cannot be regenerated at full quality without
them. `tools/crop-designer-cards.py` rebuilds every portrait from them in one
pass; see that folder's README for the roster mapping.

Image ids are resolved by a glob map in `index-BMUe7iZr.js` with a
`assets/gen/{,thumb/,tex/}<id>.webp` fallback (resolved against the bundle) for anything not in the map,
which is why these portraits sit in `public/assets/gen/` under plain
unhashed names rather than alongside the content-hashed build output. Each
one ships in the three sizes the site asks for: full 1200×1600, `tex/`
768×1024, `thumb/` 400×533.

The roster itself lives in `content-DRgEOFBw.js`. 擅長項目 bullets are stored
one string per designer with `｜` between items — the i18n layer rewrites
that separator to ` / ` in English, so the team renderer splits on either.

The team layout is overridden at the end of `index-B7RSM3hh.css`, under two
comment markers. The original grid packed three cards per row with staggered
drift and two oversized "feature" ranks, and on phones it hid the portrait on
every card but those — a treatment that suited generated art better than ten
real faces. Now every designer takes a full-width row from 701px up (small
portrait left, name and 擅長項目 right) and a full-bleed portrait with the
name overlaid below that, all at the same 3:4.

Every face in the team section is now the salon's own. Numbers 1–4 are
vacant, so the roster runs 0, 5, 6, 7, 8, 10, 11, 12, 13 and 瑪利 at the
front desk.

## Patches applied on top of the snapshot

This is build output, so anything fixed here lives in a minified bundle and
will be **silently reverted by the next real build**. Each is recorded with
what to change in `src/`.

### The asset fallback path resolves against the bundle

The fallback above was `./assets/gen/…`, which resolves against the page URL,
so it only worked from `/`. A first fix made it root-absolute
(`/assets/gen/…`), which in turn broke any deployment under a sub-path such as
`https://www.mlgroup.io/at13/`: every designer portrait was requested from
`/assets/gen/…` at the host root and returned 404.

It now resolves against the module that contains it —
``new URL(`gen/…`, import.meta.url)`` — so the portraits are found next to the
bundle wherever the site is mounted. In `src/`, do the same, or build with the
right Vite `base`.

### `sizes` describes the layout the page actually has

All three rails declared breakpoints of 720px and 1200px against CSS that
breaks at 700 and 1100, and widths well under what the cards occupy, so the
browser picked a candidate one step too small and scaled it up — up to 2.7×
on the team rail before the layout changed, 2.0× on craft.

The team rail's numbers were re-measured against the current one-per-row
layout rather than carried over from the old grid, because that layout changed
every width. Each frame is now the same column the CSS defines, so `sizes`
states it literally:

| Rail | `sizes` |
| --- | --- |
| team | `(max-width: 700px) 100vw, clamp(8.5rem, 15vw, 14rem)` |
| craft | `(max-width: 700px) 100vw, (max-width: 1100px) 92vw, 46vw` |
| work | `(max-width: 460px) 100vw, (max-width: 700px) 446px, (max-width: 1200px) 44vw, 36vw` |

The team value mirrors `.team__card`'s own `grid-template-columns`, so it
tracks the root font size instead of guessing a pixel figure; Chromium
resolves it to the measured frame width exactly, 136 / 154 / 216 / 224px
across the range. The work rail needs its fixed `446px` step because its cards
stop growing between 461 and 700px of viewport — as a percentage that band
runs 97vw down to 64vw, which no single `vw` value covers without badly
over-declaring at the wide end.

Measured at sixteen viewport widths from 360 to 1920, every rail now declares
a width at or above what it renders.

### The mid variant's srcset descriptor

Each variant is made by capping the **long** edge at 1024: 1024 wide for a
square source, **768** for the 3:4 portraits — which is now every photograph
in the team section. The descriptor said `Math.min(w, 1024)` regardless, so
every portrait advertised its mid variant as `1024w` when the file was 768px
across. A 33% overstatement, on the strength of which a browser settles for
the mid variant exactly where it needs the full one. It is now derived the way
the variant is. Verified by measuring the file behind every candidate of every
rail's srcset: all nine agree.

### Content

Sunny, Wenny, 嘎嘎, 七七 and 垣垣 carried the placeholder spec, which renders
as one generic line, while the other four designers had the real 擅長項目 from
their cards. All five now carry the six-item list their own cards give, in the
same `｜` form.

The studio stat said 11 designers. The roster is nine plus 瑪利 at the front
desk; it had not moved since four designers left.

## Not indexable, on purpose

Both `public/robots.txt` and the `<meta name="robots" content="noindex,
nofollow">` tag in `index.html` block search engines. Per the comments left in
the source, this is a design proposal carrying AT13's real address and phone
number but generated imagery rather than the salon's own photography. The
designer portraits are now the salon's own; the work gallery, salon interiors,
harbour and mood images are still generated. Lift both only once the owner
approves and the rest of the imagery is real too.

## Local preview

Any static file server works for a quick look:

```bash
python3 -m http.server 8788 --directory public
```

For behaviour identical to production, including the single-page-application
fallback for unknown paths, use Wrangler (requires Node.js):

```bash
npx wrangler dev
```

## Deploy

Pushing to `main` publishes automatically, via
`.github/workflows/deploy.yml`. It can also be run by hand from the Actions
tab.

That needs one repository secret, which only an account owner can create:

1. In Cloudflare, **My Profile → API Tokens → Create Token**, using the
   **Edit Cloudflare Workers** template (or a custom token with
   *Account → Workers Scripts → Edit*).
2. In GitHub, **Settings → Secrets and variables → Actions → New repository
   secret**, named `CLOUDFLARE_API_TOKEN`.
3. Add `CLOUDFLARE_ACCOUNT_ID` the same way only if the token can reach more
   than one account.

Until that secret exists the workflow stops on its first step and says so,
rather than failing later with an authentication error.

To deploy from your own machine instead:

```bash
npx wrangler deploy
```

Confirm you are logged into the Cloudflare account that owns the
`wahmey.workers.dev` subdomain first — `npx wrangler whoami`.
