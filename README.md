# Think Tank CoWork

Website for Think Tank CoWork, Harford County's first hi-tech coworking community in Bel Air, Maryland.

**Live site:** https://toofastturnip.github.io/ThinkTank/

## Design System

"Boutique Hospitality meets Modern Creative Hub" — inspired by Bond Collective and Huckletree.

| Role | Token | Hex | Tailwind |
|---|---|---|---|
| Background base (warm linen, never stark white) | `--linen-100` | `#F9F6F0` | `bg-linen` |
| Raised surface / cards | `--linen-50` | `#FDFCF9` | `bg-linen-50` |
| Hairlines & dividers | `--linen-300` | `#E7DECE` | `border-linen-300` |
| Text primary (charcoal slate, never pure black) | `--ink` | `#2D3142` | `text-ink` |
| Body copy / muted | `--ink-500` | `#6B7183` | `text-ink-500` |
| **Primary accent** — copper/bronze (CTAs, icons, thin borders) | `--copper` | `#C8795A` | `bg-copper` / `text-copper` |
| Copper hover | `--copper-600` | `#B06546` | `hover:bg-copper-600` |
| **Secondary accent** — brand heritage baby blue | `--powder-100` | `#D4E4F7` | `bg-powder-100` |
| Baby blue section wash | `--powder-50` | `#F3F8FD` | `bg-powder-50` |
| Baby blue icon/text (accessible) | `--powder-600` | `#456FA8` | `text-powder-600` |

**Typography** — Playfair Display (editorial serif headlines, `font-serif`) + Plus Jakarta Sans
(geometric sans body/UI, `font-sans`). Scale tokens: `text-display`, `text-h2`, `text-h3`,
`text-h4`, `text-lede`, `text-kicker`.

**Conventions**
- Sections use `py-20 lg:py-28` for macro white-space, separated by `border-t border-linen-300`
  or an alternating `bg-powder-50` band.
- Every section header pairs a small uppercase copper kicker (with a thin rule) above the serif `h2`.
- Cards: `rounded-2xl`, `shadow-soft`, `hover:border-copper hover:shadow-lift`.
- Buttons are full pills: copper for primary, powder blue for secondary, hairline outline for tertiary.

## Photography

Source photos live in `images2/` as camera-original HEIC (plus one JPEG). **HEIC does not
display in Chrome, Firefox, or Edge**, and the originals are 1.3–3.8 MB each, so the site
uses web derivatives in `images2/web/` — 33 sRGB progressive JPEGs, semantically named
(`patio-lawn.jpg`, `conference-main.jpg`, `mailroom.jpg`, …). Originals are untouched.

Regenerate a derivative with ImageMagick (requires the libheif delegate):

```bash
# full-bleed / hero plates
magick images2/IMG_0706.HEIC -auto-orient -strip -colorspace sRGB \
  -resize '1600x1600>' -interlace Plane -sampling-factor 4:2:0 -quality 76 \
  images2/web/patio-lawn.jpg

# card & grid plates
magick images2/IMG_0244.HEIC -auto-orient -strip -colorspace sRGB \
  -resize '900x900>'  -interlace Plane -sampling-factor 4:2:0 -quality 80 \
  images2/web/office-private.jpg
```

`-auto-orient` is required: iPhone HEIC carries EXIF rotation that is otherwise lost when
metadata is stripped. Every `<img>` declares `width`/`height` to prevent layout shift and
`loading="lazy"` below the fold. Total derivative weight is ~5 MB, of which only the hero
(~300 KB) is eager.

Next easy win: emit WebP or AVIF alongside the JPEGs and serve via `<picture>` — roughly a
30–50% saving. Not done here to keep the page a single dependency-free file.

## Page structure

Image-led, following Bond Collective's ordering:

1. **Hero** — headline, lede left / CTAs right, then two large `rounded-4xl` photo plates
2. **Workspaces** — four photo-led cards (photo above title, perks, CTA)
3. **Amenities** — showcase photo + checklist, then a six-tile captioned photo grid
4. **Patio** — the restaurant's outdoor space next door
5. **Meeting Rooms** — one feature plate + two supporting plates, then add-on pricing
6. **Gallery** — 16-tile bento grid, click any tile for the lightbox
7. **Location** — exterior photography + embedded map
8. **Contact** — photo + tour request form
9. **CTA** — photographic band with a dark scrim
10. **Footer**

### Gotcha: Play CDN opacity modifiers

The Play CDN silently failed to emit `bg-ink-900/72` and `bg-linen/98`, leaving the CTA scrim
and the mobile menu panel fully transparent (illegible text in both). Other non-standard
opacities such as `/45` and `/15` compiled fine, so the trigger is not simply "outside the
default scale" — the runtime scanner is just unreliable here. Both are now plain CSS classes
(`.scrim-ink`, `.panel-linen`) in the `<style>` block. **Do not rely on an opacity modifier
for anything text legibility depends on** while the Play CDN is in use; a compiled build
removes this class of problem entirely.

### Tailwind build

Tailwind is loaded via the **Play CDN** so the site stays a zero-dependency static page you can
open directly. The CDN compiles at runtime and logs a production warning; the design tokens are
mirrored as CSS custom properties in `:root`, so moving to a compiled stylesheet is a drop-in swap:

```bash
npx tailwindcss@3 -i src/input.css -o dist/output.css --minify
```

Copy the `tailwind.config` object from the `<head>` into `tailwind.config.js`, then replace the
two CDN `<script>` tags with `<link rel="stylesheet" href="dist/output.css">`.

Note: the Play CDN only generates utilities it finds in the markup, so any class added
dynamically by JS must also appear somewhere in the HTML (or be written in the `<style>` block).
