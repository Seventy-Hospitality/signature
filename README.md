# Seventy Hospitality — Email Signature Generator

A simple, single-page tool for generating branded email signatures.

## Usage

Open `index.html` in a browser, choose a brand, fill in your details, and copy the signature directly into Outlook (or any email client that supports HTML signatures).

Available brand options:

- **Seventy Hospitality** — uses `@seventyhospitality.com`; green name, dotted phone format, and the legal disclaimer
- **Yumpling** — uses `@yumpling.com`; red (`#C3272E`) name and logo, `(###) ###-####` phone format, an Instagram handle, and no disclaimer

Two signature types are generated:

- **New Message** — full signature with logo, divider, and contact info (Seventy also appends the legal disclaimer)
- **Reply** — compact single-line format for replies and forwards

Brand logos and the Instagram icon are embedded as base64 PNGs (recolored from the source SVGs) so they render in email clients that don't support SVG.
