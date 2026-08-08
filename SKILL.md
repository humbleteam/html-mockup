---
name: html-mockup
description: Builds a pixel-faithful single-file HTML mockup from a reference screenshot by counting colors, photos, items, and states before writing markup. Use when a user says "turn this screenshot into HTML", "build a mockup that matches this design", "code this screen exactly", or pastes an image and asks for a coded version. Do not use for layout exploration with no reference image - use ascii-wireframes instead.
---

# HTML mockup

Turn a reference screenshot, URL, or design description into one self-contained HTML file that matches the reference exactly - same colors, same item counts, same component states. The method is count-then-render: write a census of what the reference contains before writing a single line of markup, then treat that census as a contract the render must satisfy.

## When to run this

Trigger on: a pasted screenshot with a request to code it, "build/code/turn this into HTML", "match this reference", "make a working mockup of this screen", a Dribbble/Figma screenshot with a fidelity request.

Skip and say so if: the request has no reference image and no detailed description to work from - ask for one first (see Edge cases). For early-stage layout exploration with no reference to match, point the user at the `ascii-wireframes` skill instead - this skill's job is fidelity to a known reference, not generating new layout ideas.

## Step 1: Reference census (mandatory, before any markup)

Study the reference image carefully. Before writing `<!doctype html>` or any CSS, write this exact comment block as the literal first lines inside the HTML file:

```html
<!-- CENSUS:
  PALETTE: bg=#___ text=#___ accent=#___ secondary=#___ [every distinct color, exact hex]
  TEMPERATURE: warm|cool|neutral [warm = amber/olive/tan/gold tones; cool = blue/teal/slate]
  PHOTOS: [count] regions - [list each: "hero portrait 390x500", "food circle 60x60 x3", etc.]
  ITEMS PER SECTION: [exact counts - "3 list rows", "1 transaction", "5 nav tabs"]
  CUT OFF AT EDGE: [none | each item the frame clips - "list row 4, top half visible", "card 3, left third visible"]
  ACTIVE STATES: [which element is selected + how - "Home tab: filled icon + dot indicator"]
  BUTTON FILLS: [per button: filled-dark|filled-color|outlined|ghost - "Save: filled #1A1A1A, Delete: outlined red"]
-->
<!doctype html>
```

This census is the contract for the rest of the task. Every hex value, every count, every state description written here must show up in the rendered HTML. If you render a 4th list row when the census says 3, that is a bug - go back and fix the render, not the census.

If you are working from a text description instead of an image (no reference provided), write the census anyway using your best judgment, and mark every guessed field with `[assumed]` so the user can correct it in one pass.

## Step 2: Photo regions - real placeholders, never gradient blobs

Wherever the reference shows a photograph (portrait, food, product, room, landscape), use an actual `<img>` tag pointing at a placeholder photo service. A gradient standing in for a photo is the single most common way these mockups look fake - do not do it.

- General photos: `<img src="https://picsum.photos/{w}/{h}?random={N}" alt="...">` - vary the `random` seed per image so repeated photo slots don't render identically.
- Faces / avatars: `<img src="https://i.pravatar.cc/{size}?img={N}" alt="...">`.
- Tint toward the reference's warmth with a CSS filter on the `<img>`: `filter: brightness(0.9) sepia(0.2) saturate(1.3) hue-rotate({deg});` - pick the hue-rotate degree by eye against the reference.
- Full-bleed hero photos: fill the container, `object-fit: cover`.
- Circular thumbnails (avatars, food): `border-radius: 50%; width: {n}px; height: {n}px; object-fit: cover;`.
- Never use `radial-gradient` or `linear-gradient` in place of a photo. Multi-stop gradients read as a broken placeholder at any zoom level, never as a photograph.

## Step 3: Exact color matching

1. Extract hex values from the reference by close observation, not by naming the nearest CSS color keyword. A warm dark grey (`#1A1A1A`) is not the same as a cool near-black (`#0D1A0F`); a warm off-white (`#F5F2EB`) is not the same as pure white (`#F5F5F5`).
2. Declare every extracted color as a CSS custom property on `:root`, named by role: `--bg-primary`, `--text-primary`, `--accent`, `--secondary`, etc. Role names, not value names (never `--gray-1`).
3. Use only those custom properties everywhere else in the CSS. No raw hex values outside the `:root` block. If you catch yourself typing a hex code inside a rule, stop and add it to `:root` first.
4. Keep temperature consistent: if the reference leans warm (amber, olive, tan, gold), every neutral in your palette should lean warm too. If it leans cool (teal, slate, blue), keep neutrals cool. A mismatched neutral is visible even at thumbnail size.

## Step 4: Component state fidelity

Render the exact visual state shown in the reference for every interactive element - do not default to a generic button style.

- Filled buttons (solid background, contrasting icon or label): `background-color` set to the exact fill color. A solid dark circular button with a colored icon is not the same component as a transparent circle with a colored border - render what the reference shows.
- Outlined buttons: `border: 2px solid` with a transparent background, no fill.
- Active navigation tab: must differ from inactive tabs by at least two signals at once - for example filled vs outlined icon, brand color vs grey, plus an indicator dot or underline. A single color change alone often reads as "same state, different photo."
- Floating action buttons: elevated with `box-shadow: 0 4px 12px rgba(0,0,0,0.3)` minimum, positioned overlapping the edge of whatever it sits above, sized 56-64px diameter.
- Notification dots: 8-10px circle, reference-matched color (commonly a red like `#FF3B30`), with a 2px ring in the surrounding background color.

## Step 5: Content count lock

Render exactly the number of items the census counted. Three rows in the reference means three rows in the output, never four "to look fuller" and never one collapsed down to save space. If the reference has visible empty space below the content, that space is intentional - preserve it rather than stretching items to fill the viewport.

An item the frame cuts in half is neither empty space nor a fourth item. It is the reference telling you the list scrolls, and it is the one place the count lock has to be read carefully: count fully visible items and clipped items separately in the census, then render both.

- Render the clipped item clipped, cut where the reference cuts it - put the list in a container that overflows the frame rather than trimming the markup. The cut lands inside the item, not in the gap between two items.
- Never complete a clipped item into a full row. That adds an item the reference does not show and turns a scroll cue into a longer list.
- Never drop it either. Three rows and a half rendered as three rows leaves dead space underneath and reads as a short screen instead of a scrolling one - the same failure as padding, pointing the other way.
- A clipped item follows every other rule unchanged: same placeholder photo sources, same palette, same states as its full siblings.
- Empty space stays empty only when the reference shows nothing at all below the last item.

## Icons

- Inline SVG paths only, from Lucide, Heroicons, or Phosphor. No external icon-font CDN, no emoji.
- Match the icon's metaphor to its label, not just its category: an account/wallet icon should look like a card or a bank, not a generic grid; a transfer icon should be a bidirectional arrow, not a refresh icon.
- Look at the reference's actual icon shape before picking a name from an icon set - the silhouette matters more than the label text next to it.

## UI gradients (backgrounds, headers - not photos)

Sample at least 3 stops from the reference gradient. A 2-stop `linear-gradient(brand, black)` hits black far faster than most real gradients do and reads as flat. Example: `linear-gradient(180deg, #1A6B5E 0%, #14584D 40%, #0D4A3F 70%, #082F28 100%)`. Match the gradient's temperature to the rest of the palette.

## Brand logos without an available SVG

Render the brand's first letter, white or dark depending on contrast, centered in a rounded square filled with the brand color (8px corner radius, 36x36px default size).

## Default dimensions

Use these unless the request or reference specifies otherwise:

- Mobile: 390 x 844px (content height may grow, never shrinks below 844).
- Tablet: 768 x 1024px.
- Desktop: 1440 x 900px.

Mobile is the default when the target device is unstated.

## Output format

Deliver one self-contained `.html` file: a full `<!doctype html>` document with an inline `<style>` block (no external stylesheet, no build step, no framework), the census comment as the first lines, and CSS custom properties on `:root`. The file must open correctly in a browser with no other files present.

## Edge cases

- No reference image and no detailed description: ask for a screenshot before proceeding. If the user wants to move forward anyway, fall back to a described layout, but still write the census first with every guessed field marked `[assumed]`.
- Multiple distinct screens in one image: ask which one to mock up rather than guessing or merging them.
- Reference font is unclear or a custom font you can't identify: use the nearest system font stack (e.g. `-apple-system, "Segoe UI", Roboto, sans-serif`) and note the substitution in a code comment near the `font-family` declaration.
- Reference is blurry or too small to extract exact hex values: say so, extract your best estimate, and mark the uncertain palette entries in the census rather than presenting a guess as measured fact.
- Frame is an arbitrary crop rather than a device screen: a device screenshot's bottom edge is the viewport, so an item cut there means the content scrolls. A crop's edge is wherever someone dragged the selection box and means nothing about scrolling. When you cannot tell which you are looking at, treat it as a device viewport, write the assumption into the census next to the cut item, and say so on delivery.
- Content clipped horizontally (a carousel card peeking off the right edge, a wide table): same rule as a vertical cut - render the peek at the width the reference shows, never rounded up to a full card or dropped.
- Reference shows a scrollbar or scroll indicator but no partially visible item: record it in the census as a scroll cue, render the indicator, and do not invent extra rows to justify it.

## Before you deliver: self-check

Re-read your own census against your own render. For each line in the census, find the matching value in the HTML. If any census line has no matching render output - a color, a count, a state - fix the render before sending it. A mismatch between the census and the render is the single failure mode this skill exists to prevent.

Check the CUT OFF AT EDGE line at the census dimensions specifically: every item listed there is still clipped in the render, the cut falls inside the item rather than between two of them, and none has quietly grown into a whole one.

## Reference material

See [references/census-checklist.md](references/census-checklist.md) for the full census-field checklist and the state-fidelity rules in a single scannable list - read it when you want a compact pre-flight check before delivering a mockup, or when auditing someone else's mockup against a reference.
