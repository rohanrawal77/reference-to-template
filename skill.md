---
name: reference-to-template
description: Turn reference images into reusable HTML/CSS templates, then offer 3 distinct design directions to choose from before refining. Analyzes uploaded design references (colors, fonts, layout, elements, style), asks which elements the user likes, synthesizes across multiple screenshots, builds one faithful recreation plus two alternate creative directions on the same content, and produces a single-file HTML/CSS template with swappable content and a live edit panel. Use whenever the user uploads a design reference, screenshot, social graphic, post, or ad and says anything like "make me a template from this", "recreate this design", "extract this style", "copy this layout", "I want something like this", or "steal this design". Also trigger when the user shares images of posts they admire and wants to reuse the look, mix elements from several references, or apply their brand kit to a design they like, even without the word "template". Keep using this skill for follow-up tweaks and variations on a template built earlier in the conversation.
---

# Reference to Template

Take one or more reference images, extract their design DNA, let the user pick the elements they like, then build a faithful recreation plus two alternate creative directions, all sharing the same content zones. The user picks a favorite, then iterates on it with a live edit panel.

The user is typically non-technical. They direct; the skill builds. Never assume they will edit CSS by hand beyond the panel's controls.

Specialized, environment- or input-conditional workflows live in `references/` and are loaded only when that path is actually taken:
- **`references/design-reference.md`** — platform dimensions, font matching, brand-kit color derivation, `ui-ux-pro-max` cross-reference. Load when doing Step 1's platform match, Step 3's font pick, or a single-hex-code brand kit.
- **`references/image-embedding.md`** — base64 embedding pipeline. Load only when the user provides their own photo/screenshot to place in the template (never for reference-image assets — see Guardrails).
- **`references/claude-code-mode.md`** — sub-agent build/audit loop and automated Playwright PNG export. Load only when running inside Claude Code.

## Workflow

### Step 1: Analyze the reference(s)

For each image, extract: palette (hex codes + rough ratios), typography (weight, case, size hierarchy, closest Google Font — see design-reference.md), layout (grid, alignment, content zones), a numbered element inventory (borders, badges, highlights, shadows, icons, dividers — anything distinct), and likely platform from aspect ratio (design-reference.md has the table).

Name **the one thing that makes it work** — the single move carrying the design (a giant number, a color clash, a marker highlight). This must survive every variation built in Step 3.75.

For multiple references, build one combined inventory: what they share becomes the default system, what differs becomes a Step 2 choice.

### Step 2: Ask what they want

Summarize what was extracted in a few lines, then ask up to four questions (skip any the conversation already answers):

1. **Elements**: which extracted elements to include (multi-select, grouped by image if multiple references).
2. **Colors**: keep the reference's exact palette, or swap to a brand kit?
3. **Format scope**: just this one post, or a small set sharing the same system (quote version, list version, etc.)? This is about content variety, separate from the three design directions in Step 3.75.
4. **Platform/size**: only if the aspect ratio is ambiguous.

### Step 3: Build the faithful recreation

One self-contained HTML file, flow layout inside the canvas (flexbox column — header, headline, subhead, media, footer each push the next down; never fixed pixel `top` offsets, since a zone sized off the reference's proportions can overflow into a fixed neighbor). CSS variables in `:root` for every color, font, and key size, on an 8/16/24/32/48/64px spacing scale, with plain-English comments. Google Fonts only, named honestly if substituting. Placeholder text in the user's actual domain, wrapped in `<!-- SWAP: name -->` comments. Only the elements picked in Step 2. Never reproduce logos, watermarks, or another person's photo from the reference — see Guardrails.

**Self-check before moving on** — verify against the actual code, fix and re-check until all pass:
- No text overflow at both short and long placeholder lengths.
- Headline-to-canvas proportion matches the reference, not just its raw pixel size.
- Zone spacing is non-negative everywhere (flow layout should guarantee this — confirm it did).
- Text levels are clearly differentiated in weight/size (>15% apart).
- Readable contrast throughout.
- The Step 1 signature move is still dominant.
- Fully visible with no horizontal scroll at ~800px and ~1400px widths (panel must float, never share layout width with the canvas — see Edit Panel Spec).

### Step 3.75: Generate two alternate design directions

Always run this after the recreation passes its self-check. Build two more versions of the same content — same zones, same order, same copy — each taking a real aesthetic risk: different palette, font pairing, and/or layout treatment. Pull from the `ui-ux-pro-max` skill's palette and font-pairing library (design-reference.md has the cross-reference) rather than inventing from scratch. The structural rules from Step 3 (flow layout, CSS variables, spacing scale, no-overflow) apply to all three — only the aesthetic choices are free to diverge.

Present all three as tabs or stacked canvases in one file. Ask which one to carry forward in a single short question — don't write a paragraph per option.

### Step 4: Deliver

State briefly what the self-check caught (if anything) and any honest remaining deviation from the reference. Give a short swap guide in chat: which variables are brand colors, which blocks are text, how the edit panel works (click to select, edit text inline, Copy Changes to hand back tweaks), and how to export — hit **True Size** to disable the auto-fit scale, then screenshot at real pixel dimensions (or see claude-code-mode.md for automated export). Then stop.

### Step 5: Iterate

Stay in this skill for anything that comes back: brand-kit swaps (`:root` variables only), small tweaks (apply directly, re-run the relevant self-check bullet if it touches sizing), new format variations (reuse the system, don't redesign), and pasted Copy Changes diffs (apply exactly, don't reinterpret). Iterate on the version the user picked in Step 3.75 unless they name another.

## Edit Panel Spec

Every template ships with this. It's the only JavaScript in the file.

**Layout**: the panel is `position: fixed`, never a flex/grid sibling of the canvas — a shared-row panel adds to total page width and forces horizontal scroll, which can clip the canvas out of view in constrained frames. The canvas sits in a scaling stage (`transform: scale`, recalculated on load and resize against available viewport width) so it's always fully visible. A **True Size** toggle disables the scale for exact-pixel screenshots. A **Hide Panel** toggle collapses it for clean captures.

**Editing model**: constrained to the design system, never free-form. Global controls (always visible): one color input per `:root` variable, a font dropdown (2-3 preloaded options), size sliders for the key variables — but watch for accidental entanglement: if two unrelated elements share one `:root` size variable because they happened to start at the same size, that global slider silently resizes both together with no way to separate them. Only share a variable between elements meant to always move in lockstep. Click-to-select on canvas elements (`data-name` attribute) shows just that element's controls: a **per-element font size** and **letter spacing** override (independent of any global slider — derive each element's slider min/max from its own `getComputedStyle` default, e.g. 0.3×–3×, rather than one fixed range for every element, since a 12px label and a 140px headline need very different scales), a **per-element text color** override (independent of the global palette — lets one word/line carry a different color than the rest), show/hide, and X/Y position. `contenteditable` text blocks for all SWAP content; clicking one both selects it (panel controls appear) and places the text cursor, so repositioning/recoloring/resizing and text-editing happen in the same click.

- **Split multi-line text into sibling `contenteditable` spans joined by a `<br>` between them — never a `<br>` inside one field.** Diffing `.textContent` across an internal `<br>` silently drops the line boundary (two edited lines can merge with no space: `"Site. Brand." + "Data."` → `"Site. Brand.Data."`), and one field can't carry two colors anyway. Splitting fixes both.
- **Free dragging, not just nudge sliders**: click briefly to select; move the pointer past a ~4px threshold before release to drag-reposition live instead (divide pointer-delta by the stage's current scale factor first). Store the offset in `data-x`/`data-y` attributes and drive `transform: translate(...)` from those — exact and diffable, not regex-scraped back out of a transform string. Round to whole pixels on release; keep the X/Y sliders as a synced fine-tune companion, not a separate ±120px-only mechanism.
- **Gotcha**: nested `[data-name]` elements (a word inside a headline block inside a header row) each get their own pointer listeners. Without `ev.stopPropagation()` at the top of `pointerdown` (not just `pointerup`), the event bubbles and every ancestor also calls `setPointerCapture` — the last one (an ancestor, since bubbling fires child-first) wins, so a drag on a child silently moves its parent instead. Stop propagation on pointerdown.

**Copy Changes**: diffs current state against a load-time snapshot (read color/font-size/letter-spacing defaults per-element via `getComputedStyle`, not assumed from a shared CSS variable, since a per-element override reads back from the element itself), outputs only what changed (`--accent: #FF5C39 → #1A3AFF`, `headline / text: "old" → "new"`, `headline-line2 / position: x 30 y -15`, `headline-line2 / color: #1A3AFF → #FF0000`, `kicker / font size: 23px → 40px`), copies to clipboard with a visible textarea fallback (clipboard access can fail in sandboxed frames). **Reset All** restores the snapshot. No localStorage/sessionStorage — not available in artifacts; the diff is the only save path.

## Guardrails

- Recreate design systems and layouts; never reproduce another brand's logo, mark, watermark, or a real person's photo from the reference. Use a labeled placeholder box for photos, with the reference's crop/treatment applied in CSS.
- If the reference is a specific brand's ad or copyrighted illustration, recreate the structural style and say plainly the output is inspired-by, not a copy.
- If an image is too low-resolution to read reliably, say so rather than inventing details.