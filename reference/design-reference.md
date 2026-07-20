# Design Reference

Load this file during Step 1 (platform match), Step 3 (font pick), or when deriving a brand kit from a single hex code.

## Platform Dimensions

| Platform / format | Pixels | Aspect |
|---|---|---|
| Instagram / LinkedIn square | 1080 × 1080 | 1:1 |
| Instagram / LinkedIn portrait, or carousel slide | 1080 × 1350 | 4:5 |
| Instagram story / Reel cover | 1080 × 1920 | 9:16 |
| LinkedIn / X landscape | 1200 × 627 | ~1.91:1 |
| X post image | 1600 × 900 | 16:9 |
| YouTube thumbnail | 1280 × 720 | 16:9 |

If the reference's aspect ratio matches none of these, ask the user.

## Font Matching

Identify the closest free Google Font by classification:

- Heavy condensed display (Druk, Tungsten vibes) → Archivo Black, Anton, Oswald
- Geometric sans (Futura, Circular vibes) → Poppins, Montserrat, DM Sans
- Grotesque / neo-grotesque (Helvetica, Inter vibes) → Inter, Roboto, Work Sans
- Humanist sans (Gill Sans vibes) → Nunito Sans, Source Sans 3
- Modern serif (editorial, Vogue vibes) → Playfair Display, DM Serif Display
- Classic serif (book vibes) → Lora, Source Serif 4, EB Garamond
- Handwritten / marker → Caveat, Permanent Marker, Shadows Into Light
- Mono / techy → JetBrains Mono, Space Mono, IBM Plex Mono

State the substitution in a CSS comment. Never claim to have identified the exact commercial font unless certain.

**Cross-reference**: if a reference's typeface doesn't cleanly fit a category above, or a Step 3.75 variation needs a bolder pairing than this short list offers, pull from the `ui-ux-pro-max` skill's 57 font-pairing library instead of guessing further. Same applies to its 161 color palettes for Step 3.75's alternate directions when the reference's palette alone doesn't suggest a strong second option.

## Brand Kit: Single-Color Palette Derivation

Use when the user gives one hex code and no fuller kit, rather than asking for more codes up front:

- Lightened variant (~20% lighter) — secondary accents, tags on dark
- Darkened variant (~30% darker) — text-on-color, gradient anchor
- Tinted off-white — light backgrounds (never pure #fff; warm primary → warm cream, cool primary → cool gray-white)
- Near-black with a subtle tint of the primary — dark backgrounds

Confirm the derived palette in one line before building: "derived a full palette from your teal — lighter mint for tags, deep teal for text, warm-white background. Good, or adjust?"