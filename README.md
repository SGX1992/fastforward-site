# FastForward Global — website

Static site for **fastforward.global**, repositioned around the *house of brands* story:
strategic partners & alliances, event experiences (DDX), and ventures.

No build step. Open `index.html`, or serve the folder:

```bash
python3 -m http.server 8777
```

## Structure

```
index.html            # the whole page
assets/css/style.css  # all styles (linked with ?v=N for cache-busting)
assets/img/           # images pulled from the existing Squarespace site
```

## Brand tokens

Sampled from the **live** site rather than eyeballed — see "Where things came from" below.

| Token | Value |
| --- | --- |
| Background | `#0E0E0E` |
| Ink | `#FFFFFF` |
| Accent | `#1A3CF7` — the one blue; the original mint `#00FFD7` was retired on 2026-09-12 (two-colour feel). On near-black it is ~2.4:1, so accent *text* is neutral white with the blue on lines/chips/fills |
| Display / body | Helvetica Neue → Helvetica → Arial (system stack, as on the original site; not a webfont, so Windows/Android see Arial) |
| Labels & eyebrows | Helvetica Neue too, uppercase and wide-tracked (`--label`); the page loads **no external fonts** |

## The hero annotation marks

The circle, wave and scribble in the headline are reproductions of the Squarespace
"text highlight" feature the current site uses. The config was read out of the live
page (`TextAttributes-props` JSON):

- `explore` → **circle**, accent colour, round linecap
- `digital experiences` → **wave**, white, square linecap
- `innovations` → **scribble**, white, square linecap

All three animate left-to-right on load.

**They are animated with a `clip-path` wipe, not a `stroke-dasharray` draw.** This is
deliberate: the marks stretch to the width of their text via `preserveAspectRatio="none"`,
and they use `vector-effect: non-scaling-stroke`. Dash lengths are measured in *user*
units while a non-scaling stroke renders in *screen* space, so the two disagree and the
dash only ever covers part of the path. Don't "fix" this back to a dash animation.

## Motion

All of it is vanilla CSS + JS, no libraries. Everything is disabled or flattened under
`@media (prefers-reduced-motion: reduce)`.

**Everything reveals.** A JS pass tags every eligible element (`.eyebrow, h2, h3, p,
blockquote, .btn, .metric, .contact__mail, .contact__meta, .event__hero`) with `[data-rv]`
and an index **within its section**, so each block cascades rather than the section
appearing at once. The hero, the gallery tiles and the city names are excluded — they have
their own choreography. To opt an element out, keep it inside `.tile` or `.event__hero-in`,
or drop it from `SEL` in the script.

> Careful with that selector: `'.parent ' + SEL` does **not** scope a selector list — only
> the first selector gets the prefix, so a line like that silently matches (and untags)
> most of the page. Scope each selector individually or filter with `.closest()`.

**Images** wipe open with a `clip-path` reveal while the picture inside un-zooms from
1.16 → 1 and comes up out of a slight desaturation, staggered per tile via `--n`. On hover
a tile lifts, the image scales and goes to full colour, and a single sheen crosses the
frame. Hover effects are gated to `hover:hover` + `pointer:fine`.

**On load** (orchestrated, ~1.5s): nav drops in → logo rises → headline reveals word by
word → the three annotation marks draw → subline and scroll cue fade up. The headline is
split into `.w` spans by JS at runtime; **the `.mark` spans are kept atomic** so a mark
is never torn away from its word. Nothing is hard-coded — re-word the `<h1>` freely.

**On scroll**, driven by a single rAF-throttled handler:

- a mint progress hairline under the nav (`--p` on `.nav__prog`)
- the hero background sequence cross-fades on its own 32s cycle
- parallax on every image tagged `data-px` (JS sets `--py`, CSS composes it with the
  hover scale so the two never fight)
- section reveals + a staggered cascade across the alliance cells
- the metric numbers count up, with a mint underline sweeping each cell

**On hover**: brand cards tilt toward the cursor (max ~5°, `hover:hover` + `pointer:fine`
only), images scale, links sweep an accent underline.

### Notes for whoever edits this next

- **Parallax targets are selected by `loading="lazy"`.** That attribute is on every
  content image and on none of the logos, which is exactly the set that should move. If
  you add an image that must *not* parallax, leave `loading="lazy"` off it or strip its
  `data-px`.
- **Transforms are composed through CSS custom properties**, never by writing
  `el.style.transform`. Parallax sets `--py`, tilt sets `--rx`/`--ry`/`--ty`. Setting
  `transform` inline directly would clobber the hover states.
- Logo images (`.brand__logo`, `.ddx`) need selectors that out-specify the
  `object-fit: cover` media rules around them, or they stretch to fill the card.
- **Verifying motion needs a *visible* tab.** Browsers suspend rAF, CSS transitions and
  scroll events in hidden/background tabs, so an automated check against a headless or
  backgrounded pane will report the animations as dead when they are fine.

## Live

**https://fastforward.global/** — launched 2026-09-12. GitHub Pages with the `CNAME` file set
to the apex; `www` redirects to it; HTTPS enforced. DNS is at GoDaddy: `@` A records to
GitHub's four IPs, `www` CNAME to `sgx1992.github.io`, mail records untouched. To ship a
change: `git pull` first (GitHub commits the `CNAME` file), edit, push to `main`.

## Test URL

**https://sgx1992.github.io/fastforward-site/** — GitHub Pages, served from the `main`
branch of `github.com/SGX1992/fastforward-site` (public repo; free-plan Pages requires it).
HTTPS is enforced. Push to `main` and it redeploys in about a minute.

This is a staging URL for sharing, not the launch: the production plan is still to point
`fastforward.global` at a host (see the GoDaddy notes) — GitHub Pages works for that too
via a `CNAME` file, if you'd rather not add Netlify.

## The showreel

`assets/video/hero.mp4` — **already in the repo, nothing to place.** It was pulled from the
live Squarespace site, which serves it as HLS rather than a file you can download directly:
the page exposes an `alexandriaUrl` whose `{variant}` placeholder resolves to
`playlist.m3u8`, and the segments were remuxed to MP4 (`-c copy`, so no re-encode).
11.9s, 1920x1080, h264/aac, 2.4MB.

It lives in its own **`.reel` band between the positioning section and the gallery**, not
behind the hero. As a hero backdrop it had to be graded down so hard (a measured mean luma
of 12/255) that it was effectively invisible; as a subject it keeps almost all its own
light (`contrast(1.04) saturate(.92)`). The hero uses the cross-fading stills instead.

**The frame can never be empty, and the clip can always be started.** Two rules, both
learned from a real "video not visible" report:

- The `<video>` is **never under a `clip-path`** (WebKit may not paint it), and it does
  **not** draw its own poster — a real `<img class="reel__poster">` sits underneath. The
  reveal wipe is an `::after` curtain that slides away.
- State follows **media events**, not intent: `.is-playing` is set on `playing` and removed
  on `pause` / `error`; while it is absent, the poster and a **"Play showreel" button**
  show, and the button starts playback from a user gesture (allowed everywhere).

**Every reveal on the page fails open.** Scroll-triggered reveals hide content until an
IntersectionObserver adds `.is-in`. That must never be the only way content appears, so
zero-duration delayed CSS animations force the final state regardless: the showreel curtain
lifts by 3s after load, everything else is visible by 7s. Same end values as the `.is-in`
path, so normal motion is unchanged — it just cannot leave anything hidden. Keep this when
adding new reveals.

It **plays only while on screen** — an IntersectionObserver starts it at 25% visibility and
pauses it on the way out, so a below-the-fold video costs nothing until someone scrolls to
it. `preload="metadata"` for the same reason. The `play()` call is nudged and its rejection
swallowed, because some browsers (iOS Low Power Mode, battery saver) refuse programmatic
playback; if refused, the poster stays put.

> **Serving it locally:** `python3 -m http.server` does **not** implement HTTP Range, so
> the video will silently never load. Use `npx serve` instead.

## Page structure

A **single landing page — no subpages, and nothing on it links to one.**
Hero → positioning → gallery → DDX → mission → contact.

**Hero.** Full-bleed cross-fading stills (`.hero__seq`, 4 images, 32s cycle) under a
scrim, with the annotated headline over it. There is **no video on the current
fastforward.global to reuse** — the only hits there are Squarespace
`sqs-video-background-native` *class names* with no media behind them. To drop in a real
film later, replace `.hero__seq` with the `<video class="hero__video">` block commented
inline in `index.html`; the grade and sizing already account for it. Keep it `muted` +
`playsinline` or iOS blocks autoplay.

**Gallery.** A wall of work, deliberately *not* an index: no links, no captions, nothing
singled out.

*Why every column ends flush:* all three columns carry the **same multiset of aspect
ratios**, just rotated (`4/3, 3/4, 1/1, 16/9, 4/5, 3/2`). Identical totals means identical
column heights, but because the rotation differs, rows never line up — so it still reads
as a gallery rather than a grid. Verified: all three columns measure 2786px and end on the
same pixel.

*Why the frames never move:* depth comes from parallaxing the **image inside its frame**
(the `img` is 120% tall and slides within `overflow:hidden`), never the column. Moving the
columns would break the flush bottom edge. Amplitude per column comes from the column's
`data-speed`, so the three still read at different depths.

> Do **not** put `will-change: transform` back on `.gallery__col` or `.tile`. Held
> permanently on a tall element it forces a compositor layer for the life of the page and
> made the columns drop out of paint.

**DDX backdrop — one file to swap.** The section's background is
`assets/img/ddx-hero.jpg`. Replace that single file and nothing else needs editing; it is
currently a copy of `ddx-stage.jpg` standing in until the intended photograph is supplied.

**DDX.** Leads on "one of the world's fastest growing UX and design conferences", then a
strip of conference photography, then the "Learn more about DDX" button, plus **upcoming-edition tiles** linking to each
edition's page on ddxconference.com (San Diego, Miami, Tokyo as of 2026-09-12, and an
"All editions" tile). These are hand-maintained — when the calendar moves, update them in
`index.html`; past editions are not shown.

The **seven cities** (Tokyo, Dubai, Munich, London, Miami, New York, San Diego) now sit
under the 10 / 20 / 7 metrics strip instead, so the "7" is immediately concrete. They fit
one line at desktop.

The DDX photographs (`assets/img/ddx-*.jpg`) were taken from **ddxconference.com**, DDX's
own site — same organisation, so they are FastForward's own images to reuse. Sourced:
stage, audience, quote wall, networking, brochure.

**Topbar** is the logo plus two actions — "Think Tank" (→ ddxconference.com, new tab) and
"Get in touch" (mail, accent-outlined as the primary). It deliberately carries **no section
anchors**: this is one page and the nav should not imply navigation. Below 430px the Think
Tank pill is hidden — the same link is still in the DDX section.

**City list.** Spacing is set in `em` so the separator gap and the gap between cities are
one and the same, which keeps the row reading as a single typographic block instead of a
spaced-out list. A small JS pass (`markRowEnds`) tags whichever city ends a visual row with
`.is-rowend` and hides its separator dot, so a dot never dangles at a line break — CSS
alone cannot detect wrap points. It re-runs on resize and after fonts load.

## Gallery tint

Tiles rest as a **blue duotone** (`grayscale(.9)` + a `mix-blend-mode: color` overlay of
`--blue` at .5) and lift to full colour on hover. This is deliberately *not* removed under
`prefers-reduced-motion` — a static tint is not motion (an earlier rule did strip it there, which
is why the tint "disappeared" for some viewers).

A `color` blend takes its luminance from the photo, so a source image that is itself almost
black (the SiteWasp drone on black) reads as a flat dark block at rest — it only shows in
colour on hover. Prefer mid-tone imagery for new tiles, or give a dark one a lighter grade.

## Adding to the gallery

Tiles accept **images, GIFs and video**. A still or GIF is just an `<img>`; a clip is:

```html
<figure class="tile" style="--ar:16/9;--n:0">
  <video muted loop playsinline preload="metadata" poster="...">
    <source src="assets/video/example.mp4" type="video/mp4">
  </video>
</figure>
```

Video tiles inherit the framing, grade, hover and in-frame parallax automatically, and
play only while on screen (same observer as the showreel).

**Keep the columns equal.** Every column carries the same multiset of aspect ratios,
rotated — that is what makes the grid end flush. If you add tiles, add the *same number*
to each column and keep the ratio multiset identical across all three, or the bottom edge
goes ragged again.

Prefer MP4 over GIF for anything longer than a second or two: the GIFs in the portfolio
folder run 1–17MB each, where the same clip as h264 is a fraction of that.

## Careers

Two components in one band, sitting between the mission and the contact CTA:

| | Left | Right |
| --- | --- | --- |
| Card | Live role | Open application |
| Style | solid | dashed, unfilled |
| Action | posting URL, new tab | `mailto:` with a pre-filled subject |

The live role links to its LinkedIn posting. Roles are tracked in the **Open Roles**
Notion database (under the FastForward Global page). Note the site is a **static page and
does not read from Notion** — when a role opens or closes there, update the card in
`index.html` to match. `Status` is the source of truth; only *Open* roles belong on the
site, and `Summary` is written to be used verbatim as the card copy.

"Initiativbewerbung" is rendered as **"Open application"**, the usual English careers term
("Speculative application" is the UK variant if you prefer it).

Card heights are matched by flex, with the gap on the paragraph and `margin-top:auto` on
the button, so both buttons sit on one baseline however long the copy runs — don't set a
fixed height.

## Blue surfaces

**Current state:** one flat blue, `--blue: #1A3CF7`, and it is also `--accent` — marks, rules,
buttons, contact, footer, hover tints, the gallery duotone. A blue→teal gradient was tried and rejected. Mint stays the accent.

`--blue: #1A3CF7` is a **FastForward addition, not a DDX colour** — DDX's own palette is black / white / yellow `#FFF204` / neon greens. It carries the
contact section, the footer and the top-right primary action (solid blue, white on hover).
The city clocks use DDX's yellow `#FFF204`.

## City strip

Each city is a link to its edition page on ddxconference.com. Above the name is a
**landmark icon** (fetched with `better-icons`, mostly the `mingcute` line set) that flips to
the country's circle-flag SVG on hover; the name gets a straight highlighter-marker sweep.
Icons per city are listed in `/tmp/icons_picked.txt` at build time and inline in the markup —
swap any one by replacing its `<svg class="ico-pin">`.

## City photos on hover

The same seven Unsplash photos back three hover moments: the **city strip** (photo fades
behind the whole positioning section, tinted by the gradient), the **upcoming-edition tiles**
(photo inside the tile), and the **Explore DDX tour** below. Strip and tile images carry
`data-src` and are attached on first pointer approach, so they cost nothing on page load.

## Contact slideshow (Unsplash)

Hovering **Explore DDX** tours the seven cities in the section background — one frame every
1.5s, blue-tinted via `mix-blend-mode: luminosity`, fine pointers only. Images are hotlinked
from `images.unsplash.com` (`?auto=format&fit=crop&w=1600&q=70`, verified 200/image at
build time). Unsplash's licence permits this without attribution, but they ask for it, so the
footer carries a credit and the photos are:

- tokyo: https://images.unsplash.com/photo-1513407030348-c983a97b98d8
- dubai: https://images.unsplash.com/photo-1512453979798-5ea266f8880c
- munich: https://images.unsplash.com/photo-1595867818082-083862f3d630
- london: https://images.unsplash.com/photo-1513635269975-59663e0ac1ad
- miami: https://images.unsplash.com/photo-1605723517503-3cadb5818a0c
- newyork: https://images.unsplash.com/photo-1496588152823-86ff7695e68f
- sandiego: https://images.unsplash.com/photo-1514939775307-d44e7f10cabd

If Unsplash ever changes hotlinking terms, the images are decorative and can be dropped —
the section works without them. For production polish, self-hosting downscaled copies is the
safer option.

## Positioning

FastForward Global is framed as **the company behind DDX** — a global think tank on the
future of digital experience — not as an agency and not primarily as a "partner". The
narrative order is deliberate and worth keeping if you rewrite:

1. **Hero** — the headline (kept from the original site) plus a kicker that rotates
   through the seven DDX cities.
2. **Who we are** — the company behind DDX; foresight from the stages is the product.
   Four metrics, all grounded in DDX figures from ddxconference.com (7 cities, 3,000+
   leaders, 100% sold-out, 10+ years). The city strip shows **live local time** per city.
3. **From the DDX stages** — four recurring themes. This is the thought-leadership core.
4. **In practice / Along the way** — the showreel and gallery, positioned as *evidence*
   of the foresight rather than the pitch.
5. **DDX** → **Mission** → **Careers** → **Next step**.

**Routing.** Every primary action goes to ddxconference.com: the nav's accent button,
the explorations button, and the contact section's "Explore DDX". The email is demoted to
a small "Partnerships & press" line — it is deliberately not a headline CTA.

## Motion, second pass

- **City rotator** in the kicker: fixed `min-width` so the centred line never jumps
  between "Dubai" and "San Diego"; word slides out upward, next slides in from below.
- **Live clocks** under each city via `Intl.DateTimeFormat` with the city's IANA zone
  (`data-tz`), refreshed every 20s. Miami and New York share `America/New_York`.
- **Decode** — mono labels (eyebrows, hero subline, next-edition note) settle out of
  random glyphs on reveal. Only applied to elements whose content is a single text node,
  so nothing with children is touched. Progress is **time-based** (620ms), and a
  `setTimeout` fallback forces the final text regardless — rAF is *suspended* in a hidden
  tab, but timers are only throttled, so scrambled text can never be left on screen. The
  count-up has the same guarantee for the same reason.
- **Word-by-word headings** — every `h2` and the lead paragraph are split into `.w` spans
  by the same routine as the hero (child elements stay atomic, so `<b>DDX</b>` is one
  word). `h2.is-split[data-rv]` opts the element itself out of the fade so the words carry
  the motion alone — otherwise they double up.
- **Magnetic buttons** — `.btn` and `.nav__cta` drift a few px toward the cursor via
  `--mx/--my`, fine pointers only. The transform slot is on the base rule, so any future
  transform on those elements must compose with it rather than replace it.
- `countUp` now handles `3,000+` (digit grouping preserved) and `100%`.

## Content sources

Copy is drawn from the existing site, the FastForward Notion workspace (mission
statement, business model, client list) and sebastiangier.com/projects (project
descriptions). The mission paragraph is close to verbatim from Notion.

## Needs confirming before launch

- **The seven DDX cities are presented without dates**, which is deliberate: only the
  San Diego edition (17 Sept 2026) is confirmed from correspondence. Confirm the rest of
  the city list is accurate before launch.
- **"20+ intl. design recognitions"** and **"10+ years"** — carried over from
  sebastiangier.com and the current site respectively; check they still hold for the
  company rather than the founder.

## Where things came from

- Images: downloaded from the live Squarespace CDN at `?format=2500w`.
- Accent colour: read off the rendered annotation stroke in the browser
  (`rgb(0,255,215)`), because the palette is applied at runtime and is not in the
  stylesheets.
- Logo files are white-on-transparent PNGs; they only read against a dark surface.
