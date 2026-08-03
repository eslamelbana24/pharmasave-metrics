# Escarpment Pharmasave — favicon set

Mark: option 3f — spineless strata "E" in a keyline ring, Pharmasave red `#C6093B` on white.
The ring is dropped and the bands grow at small sizes so the E stays legible.

## Files

| File | Use |
|---|---|
| `favicon.svg` | Primary scalable icon (ring state) |
| `favicon-small.svg` | Optional small-size variant, ring removed |
| `favicon-16x16.png` | Legacy browser tab |
| `favicon-32x32.png` | Legacy browser tab / bookmarks |
| `favicon-48x48.png` | Windows shortcuts |
| `apple-touch-icon.png` | 180×180, iOS home screen (full-bleed red, no transparency) |
| `icon-192.png` | Android / PWA |
| `icon-512.png` | Android / PWA, splash |
| `icon-512-maskable.png` | Android adaptive icon (mark inside 60% safe zone) |
| `site.webmanifest` | PWA manifest |

An `.ico` is no longer needed for modern browsers; if a legacy `favicon.ico` is
required, bundle the 16/32/48 PNGs with any ICO packer.

## Next.js App Router

Move the files so Next generates the tags automatically:

- `app/icon.svg` ← `favicon.svg`
- `app/apple-icon.png` ← `apple-touch-icon.png`
- `app/manifest.webmanifest` ← `site.webmanifest` (fix the icon paths to `/icons/...`)

## Plain HTML head

```html
<link rel="icon" href="/assets/favicon/favicon.svg" type="image/svg+xml">
<link rel="icon" href="/assets/favicon/favicon-32x32.png" sizes="32x32" type="image/png">
<link rel="icon" href="/assets/favicon/favicon-16x16.png" sizes="16x16" type="image/png">
<link rel="apple-touch-icon" href="/assets/favicon/apple-touch-icon.png" sizes="180x180">
<link rel="manifest" href="/assets/favicon/site.webmanifest">
<meta name="theme-color" content="#C6093B">
```
