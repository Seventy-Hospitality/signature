# Email Signature Generator

A single-page tool (`index.html`, no build step, no dependencies) that generates
branded HTML email signatures for **Seventy Hospitality** and **Yumpling**.

- **Live:** https://seventy-hospitality.github.io/signature/
- **Deploy:** GitHub Pages, auto-publishes on every push to `main` (see [Deploy](#deploy)).
- **Source:** everything lives in `index.html`. The two `*.svg` files are the
  original logo/icon art kept for regenerating the embedded images.

## Usage

Open the page, choose a brand, fill in the fields, click **Copy Signature**, and
paste into your mail client. Two variants are produced:

- **New Message** — full signature (logo, divider, contact info; Seventy also
  appends the legal disclaimer).
- **Reply** — compact one-line version for replies/forwards.

## Brands

Brand behavior is data-driven by the `BRANDS` object near the top of the
`<script>` in `index.html`. To change copy, colors, phone numbers, etc., edit
that object — you should rarely need to touch the rendering code.

### Seventy Hospitality
- Domain `@seventyhospitality.com`, green name (`#3D5341`), small badminton logo.
- Dotted phone format (`###.###.####`).
- Includes the full legal **disclaimer**.
- Per-person fields: name, title, phone, email prefix.

### Yumpling
- Red (`#C3272E`) name + logo, parens phone format (`(###) ###-####`), **no disclaimer**.
- **Only the name is per-person.** Everything else is a fixed company value baked
  into the brand config:
  - `fixedEmail: "hello@yumpling.com"` — the email field is ignored.
  - `phones: [LIC (718) 713-1839, Midtown (646) 419-6400]` — both render on one
    line, the phone field is ignored.
  - `instagram: "yumpling"`.
- Layout: name on top, then logo · divider · (email / two phones / Instagram).

## ⚠️ Legacy Outlook (Word engine) — read before changing the markup

Outlook on Windows (2007–2021 and Microsoft 365 desktop) renders HTML with the
**Microsoft Word** engine, not a browser engine. **The Chrome preview and the
GitHub Pages page are NOT a reliable proxy for how it looks in legacy Outlook.**
We burned a lot of cycles relearning this; here are the rules and the fixes that
are already implemented. Do not "simplify" them away.

| Problem in legacy Outlook | Cause | Fix used here |
|---|---|---|
| Logo / Instagram icon forced to **top** of the cell, won't center | Word **ignores `valign`/`vertical-align` on a cell that contains only an image** | Put an invisible char in the cell so Word treats it as text: `<!--[if mso]>&zwnj;<![endif]--><img …>`. Wrapped in `[if mso]` so it only exists in Outlook. Images are `display:inline-block; vertical-align:middle`. |
| **Huge/unpredictable vertical gaps** | Word ignores CSS margins and auto-inflates line-height to the tallest element on a line | Spacing only via **cell `padding`** (never margins); every text line sets **`mso-line-height-rule: exactly`** with an explicit `line-height`. |
| Multi-line / wrapping where it shouldn't | Word's `white-space:nowrap` support is unreliable | The two-number phone line uses `&nbsp;` between every space so it can't wrap. |
| Font isn't Inter | Outlook desktop **won't load web fonts** | Stack is `Inter, Arial, Helvetica, sans-serif`; Outlook falls back to **Arial**. Inter only shows in the on-page preview (loaded via Google Fonts on the generator page — that link does NOT travel with the copied signature). There is no reliable way to force Inter in Outlook short of rasterizing the whole signature (don't — breaks links/scaling). |
| Logo blurry / wrong size | — | Logos are embedded **base64 PNGs** (not SVG — Outlook/most clients don't render SVG), with explicit integer `width`+`height` so Word doesn't size them off `height:auto`. |

General Outlook-safe rules this file follows: table-based layout (no flex/grid),
all styles inline, `role="presentation"` + `border-collapse:collapse` on tables,
`mso-table-lspace/rspace:0pt`, reset `margin:0;padding:0`.

**Testing:** there is no way to render the Word engine locally (it's Windows-only).
Real verification means pasting into actual legacy Outlook on Windows and
screenshotting. If spacing still looks wrong *after* the fixes above, the next
suspect is Outlook mangling HTML **on paste** — installing the signature as an
`.htm` file in Outlook's signature folder is more faithful than pasting.

## Regenerating the embedded images

The logo and Instagram icon are stored in `index.html` as base64 PNG data URIs
(constants `YUMPLING_LOGO_SRC` and `INSTAGRAM_ICON_SRC`). To change/recolor them,
recolor the source SVG, rasterize to a transparent PNG, base64-encode, and paste
into the constant. Requires `librsvg` (`brew install librsvg`).

```bash
# Recolor the SVG fill, then rasterize at ~2x display size (transparent bg)
sed 's/fill="#000000"/fill="#C3272E"/' "Yumpling Logo.svg" > /tmp/logo.svg
rsvg-convert -w 300 -b none /tmp/logo.svg -o /tmp/logo.png   # red yumpling logo
sed 's/fill="#000000"/fill="#808080"/' instagram-logo.svg > /tmp/ig.svg
rsvg-convert -w 64  -b none /tmp/ig.svg  -o /tmp/ig.png      # gray IG icon

# Encode to a data URI (GNU base64: use -w 0)
printf 'data:image/png;base64,%s' "$(base64 -w 0 /tmp/logo.png)"
```

Display sizes are controlled by `logoWidth`/`logoHeight` in the brand config (keep
the 70×40 aspect ratio for the Yumpling logo); the PNG is rendered larger for
retina sharpness and scaled down.

Colors: Yumpling red is `#C3272E`; contact text / IG icon gray is `#808080`;
divider gray is `#c8c8c8`.

## Local preview

```bash
python3 -m http.server 8077 --bind 127.0.0.1 --directory .
# open http://localhost:8077  (hard-refresh ⌘⇧R after edits)
```

## Deploy

This folder is its **own git repo** (`github.com/Seventy-Hospitality/signature`),
separate from the main website repo (which `.gitignore`s `signature/`). GitHub
Pages serves `main` at the root, so:

```bash
git add -A && git commit -m "…" && git push origin main
```

publishes automatically in ~30–60s. No build/CI step.
