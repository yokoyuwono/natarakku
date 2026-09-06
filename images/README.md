# Images

## What's here

| File | Used by | Notes |
|---|---|---|
| `story-01-single-origin.svg` | Story 01 frame | 4:5 placeholder |
| `story-02-precision.svg` | Story 02 frame | 4:5 placeholder |
| `story-03-patience.svg` | Story 03 frame | 4:5 placeholder |
| `favicon.svg` | Browser tab | |
| `og-cover.png` | Social sharing | 1200×630, referenced by the Open Graph and Twitter tags |

The three story images are **placeholders**. They're SVG so they stay sharp at any
size, and they match the artwork the page used before, but they are not photography.

## Swapping in real photos

Each slot in `index.html` looks like this:

```html
<picture class="ph">
  <!--
  <source type="image/avif" srcset="images/story-01-single-origin-800.avif 800w, …1200w" sizes="(max-width:900px) 92vw, 44vw">
  <source type="image/webp" srcset="images/story-01-single-origin-800.webp 800w, …1200w" sizes="(max-width:900px) 92vw, 44vw">
  -->
  <img src="images/story-01-single-origin.svg" width="800" height="1000" alt="" loading="lazy" decoding="async">
</picture>
```

1. Export each photo at **800w and 1200w**, in **AVIF and WebP**, plus one **JPEG**
   fallback. Crop to **4:5** — the slot reserves that ratio, so an off-ratio image
   will be cropped by `object-fit:cover` rather than reflowing the page.
2. Name them to match the `srcset` entries above, or edit the paths.
3. **Uncomment the two `<source>` lines.** They're commented on purpose: `<picture>`
   does not fall back when a matching `<source>` 404s, so live lines pointing at
   files that don't exist would show a broken image.
4. Point the `<img>` `src` at the JPEG and update `width`/`height` to its real pixel
   size. Keep both attributes — they're what holds the layout still while it loads.
5. **Write a real `alt`.** It's empty now because a placeholder has nothing to
   describe; a photograph does. Describe what's in the shot, not the section it sits in.

Verify afterwards with the CLS check — layout shift should stay at 0.

## Regenerating `og-cover.png`

It was rendered from an HTML template through headless Chromium rather than drawn by
hand, so it can be regenerated at any size. If the headline or the founding year
changes, regenerate it so the social card doesn't contradict the page.
