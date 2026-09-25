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
  CUT OFF AT EDGE: [none | each item the frame clips, naming the edge that cuts it - "list row 4, bottom edge, top half visible", "carousel card 3, right edge, left third visible"]
  SCROLL CUE: [none | a scrollbar or scroll indicator the reference shows with nothing clipped, naming what scrolls and where the bar sits - "vertical scrollbar, page, right edge", "pill indicator, carousel row, bottom"]
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

Both services are remote, so the photo regions are the one part of the file that is not in the file. On a machine with no connection, or behind a proxy that blocks those hosts, every `<img>` renders as the browser's broken-image icon: the fake look this step exists to prevent, arriving from the other side and worse than the gradient it replaced. Nothing in the render fixes that. Every offline stand-in within reach is one the list above already rules out, and a solid block or a flat SVG shape is the gradient's failure with the gradient taken out, so the dependency is named rather than patched. Say on delivery that the mockup pulls its photos from `picsum.photos` and `i.pravatar.cc` and needs a connection to show them. Where the user says the file will be opened offline - a flight, an air-gapped review, a deck that travels - ask for the real photographs and embed them as data URIs. That is an ask and not a fallback: the skill has no image of its own to embed.

## Step 3: Exact color matching

1. Extract hex values from the reference by close observation, not by naming the nearest CSS color keyword. A warm dark grey (`#1A1A1A`) is not the same as a cool near-black (`#0D1A0F`); a warm off-white (`#F5F2EB`) is not the same as pure white (`#F5F5F5`).
2. Declare every extracted color as a CSS custom property on `:root`, named by role: `--bg-primary`, `--text-primary`, `--accent`, `--secondary`, etc. Role names, not value names (never `--gray-1`).
3. Use only those custom properties everywhere else in the CSS. No raw hex values outside the `:root` block. If you catch yourself typing a hex code inside a rule, stop and add it to `:root` first.
4. Keep temperature consistent: if the reference leans warm (amber, olive, tan, gold), every neutral in your palette should lean warm too. If it leans cool (teal, slate, blue), keep neutrals cool. A mismatched neutral is visible even at thumbnail size.
5. A gradient is one role, not one role per stop. Store the whole expression under a single custom property - `--header-gradient: linear-gradient(180deg, #1A6B5E 0%, #14584D 40%, #0D4A3F 70%, #082F28 100%);` - and write its stops on the census PALETTE line as that role's set, not as four more roles. Rule 3 has no exception for gradient stops: written inline they are raw hex in a rule, and split into `--gradient-1` through `--gradient-4` they are value names, which rule 2 forbids.
6. A color the render needs and the reference does not contain is still a `:root` token, and it stays off the census PALETTE line. The skill creates exactly one: the page ground behind a fixed frame, which exists because the mockup is centered in a browser window that the reference never had. Give it a role name, mark it, and leave the census alone - that line reports what the reference shows, and adding a color the reference does not contain would make the census wrong in the one direction it is never allowed to be.

```css
  /* Not from the reference: the surround the frame sits on */
  --page-ground: #E8E4DA;
```

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

- Render the clipped item clipped, cut where the reference cuts it - put the list in a container that overflows the frame rather than trimming the markup. The cut lands inside the item, not in the gap between two items. The frame is a real element, fixed on the axis its cut runs across, and a census with anything on the CUT OFF AT EDGE or SCROLL CUE line is what calls for one: see [Default dimensions and the frame](#default-dimensions-and-the-frame).
- Never complete a clipped item into a full row. That adds an item the reference does not show and turns a scroll cue into a longer list.
- Never drop it either. Three rows and a half rendered as three rows leaves dead space underneath and reads as a short screen instead of a scrolling one - the same failure as padding, pointing the other way.
- A clipped item follows every other rule unchanged: same placeholder photo sources, same palette, same states as its full siblings.
- Empty space stays empty only when the reference shows nothing at all below the last item.

## Icons

- Inline SVG paths only, from Lucide, Heroicons, or Phosphor. No external icon-font CDN, no emoji.
- Match the icon's metaphor to its label, not just its category: an account/wallet icon should look like a card or a bank, not a generic grid; a transfer icon should be a bidirectional arrow, not a refresh icon.
- Look at the reference's actual icon shape before picking a name from an icon set - the silhouette matters more than the label text next to it.

## UI gradients (backgrounds, headers - not photos)

Sample at least 3 stops from the reference gradient. A 2-stop `linear-gradient(brand, black)` hits black far faster than most real gradients do and reads as flat. Match the gradient's temperature to the rest of the palette, and declare it the way Step 3 rule 5 requires - the whole expression under one role name in `:root`, used by `var()` wherever it renders:

```css
  --header-gradient: linear-gradient(180deg, #1A6B5E 0%, #14584D 40%, #0D4A3F 70%, #082F28 100%);
```

## Brand logos without an available SVG

Render the brand's first letter, white or dark depending on contrast, centered in a rounded square filled with the brand color (8px corner radius, 36x36px default size).

## Default dimensions and the frame

Use these unless the request or reference specifies otherwise:

- Mobile: 390 x 844px.
- Tablet: 768 x 1024px.
- Desktop: 1440 x 900px.

Mobile is the default when the target device is unstated.

Whether that height is a floor or a fixed edge is decided by the census, not by the device - and by the edge named on the CUT OFF AT EDGE line, because a clipped item is evidence about the edge that cuts it and no other:

- **`CUT OFF AT EDGE: none`** - the height is a floor. Content may grow past it and never shrinks below it. A full-page capture of a long screen has no viewport edge in it, and forcing it back to 844px would invent a cut the reference does not show.
- **An item cut by the bottom or top edge** - the height is fixed. That edge is the viewport, which is what the clipped item is evidence of, and a document free to grow renders that item in full: the one outcome Step 5 forbids.
- **Every item on the line cut by the left or right edge** - the height stays a floor. A carousel card peeking off the right says where the screen ends sideways and nothing about where it ends below, so a full-page capture that has one keeps its floor for the same reason `none` does. Fixing the height on that evidence hides every item past 844px - items the census counted and Step 5 requires rendered, sitting in the markup where a count check still finds them.
- **Items cut on both** - the height is fixed, and the frame cuts both ways.

A scrollbar is the same evidence with no item carrying it. The `none` branch above is written for a full-page capture of a long screen, which has no viewport edge in it - and a capture with a scrollbar in it plainly has one, so `CUT OFF AT EDGE: none` with something on the SCROLL CUE line reads the cue instead, on the same one-edge reasoning: a vertical bar fixes the height, a horizontal one leaves it a floor. A bar belonging to a panel inside the page fixes that panel's height and says nothing about the frame's, which is why the census names what scrolls and not only that something does.

A cut needs an element to be cut by. "Overflows the frame" is not a property of the document, so build the frame whenever either line is not `none`: one container at the census width, `overflow: hidden`, content in normal flow inside it, centered on the page so the size of the browser window stops changing what the mockup looks like. Write the height as `height: 844px` when it is fixed and `min-height: 844px` when it is a floor - an auto height grows with the content in it, so `overflow: hidden` has nothing to cut vertically and the floor still holds. The width is a fixed size in both cases: it is the reference's own width, and a block box does not widen to fit its content the way an auto height grows, so there is no floor to write there. Do not reach for `overflow-x: hidden` with `overflow-y: visible` instead - CSS resolves the visible axis to `auto`, and the scrollbar that appears is an element the reference does not show.

A horizontal cut also needs a row that reaches the frame's edge. Content in normal flow fits the width it is given: a row of cards shrinks them below their census width or wraps them onto a second line, and both render the peeking card whole - the same failure as a document free to grow, arriving sideways. Let the row run past the frame instead: no wrapping, each item holding its census width (`flex: 0 0 280px`).

Use `overflow-y: auto` in place of `hidden` when the user asks for a scrollable prototype rather than a still image. It is the wrong tool for rendering a scroll cue: `auto` shows a bar only while the content overflows, and a census with nothing on the CUT OFF AT EDGE line counts content that stops at the frame's edge, so `auto` renders no bar at all. Never add rows until `auto` produces a bar of its own: that is Step 5's padding failure with a reason attached, and a scrollbar the reference does not show is still a visual element the render invented.

Which route renders the cue is decided by the bar in the reference, and the fact that defeats `auto` does not stop there. `overflow-y: scroll` asks the platform for its own track, and nothing overflows either way, so the track comes back with a thumb spanning its full length - and on a platform drawing overlay scrollbars, the macOS default, it paints nothing at all until something is scrolled. Reach for it only where the reference shows a bare platform track and a full-length thumb changes nothing about what the mockup says. A reference whose thumb sits at a position, a third of the track long, is showing how much of the screen is below the fold, and the only route that renders that is the styled element itself - an inset pill, an iOS-style thumb - positioned inside the frame. Its color is a color the reference contains, so it goes on the census PALETTE line like every other, unlike the page ground of Step 3 rule 6.

Write the frame's two axes separately whenever either one is not `hidden`. `overflow-y: scroll` on its own leaves `overflow-x` at its initial `visible`, and CSS resolves a visible axis to `auto` when the other axis is not visible - the same rule that already rules out `overflow-x: hidden` with `overflow-y: visible`, arriving from the other side. The frame is then free to grow a horizontal bar on the axis it exists to clip. `overflow-x: hidden; overflow-y: scroll` is the pair. A drawn indicator needs none of this: the frame keeps `overflow: hidden` and the bar is an element inside it.

## Output format

Deliver one self-contained `.html` file: a full `<!doctype html>` document with an inline `<style>` block (no external stylesheet, no build step, no framework), the census comment as the first lines, and CSS custom properties on `:root`. The file must open correctly in a browser with no other files present - no sibling stylesheet, no image folder, no build output. That is a claim about files and not about the network: the photo regions of Step 2 load from `picsum.photos` and `i.pravatar.cc`, so a machine with no connection opens the document correctly and still shows a broken-image icon in every photo slot. Name the dependency on delivery (Step 2).

When the census lists anything under CUT OFF AT EDGE, the markup carries the frame container described above, and the file renders the same at any window size.

## Edge cases

- No reference image and no detailed description: ask for a screenshot before proceeding. If the user wants to move forward anyway, fall back to a described layout, but still write the census first with every guessed field marked `[assumed]`.
- Multiple distinct screens in one image: ask which one to mock up rather than guessing or merging them.
- Reference font is unclear or a custom font you can't identify: use the nearest system font stack (e.g. `-apple-system, "Segoe UI", Roboto, sans-serif`) and note the substitution in a code comment near the `font-family` declaration.
- Reference is blurry or too small to extract exact hex values: say so, extract your best estimate, and mark the uncertain palette entries in the census rather than presenting a guess as measured fact.
- Frame is an arbitrary crop rather than a device screen: a device screenshot's bottom edge is the viewport, so an item cut there means the content scrolls. A crop's edge is wherever someone dragged the selection box and means nothing about scrolling. When you cannot tell which you are looking at, treat it as a device viewport, write the assumption into the census next to the cut item, and say so on delivery.
- Content clipped horizontally (a carousel card peeking off the right edge, a wide table): same rule as a vertical cut, applied to the other axis - render the peek at the width the reference shows, never rounded up to a full card or dropped. It fixes the frame's width, leaves the height as it found it, and needs a row that does not wrap or shrink to fit: see [Default dimensions and the frame](#default-dimensions-and-the-frame).
- Reference shows a scrollbar or scroll indicator but no partially visible item: it goes on the census SCROLL CUE line, never on CUT OFF AT EDGE - a bar is not an item, and the clipped-item checks have nothing to check on one. It still calls for the frame, and a vertical bar still fixes the height, because a full-page capture has no scrollbar in it either. Render the bar the way the reference shows it and do not invent extra rows to justify it: see [Default dimensions and the frame](#default-dimensions-and-the-frame).
- File will be opened offline, or on a network that blocks the placeholder hosts: ask for the real photographs and embed them as data URIs before delivering. Do not substitute a gradient, a solid block or an SVG shape for the remote placeholder - Step 2 rules all three out, and a stand-in that quietly looks fake is worse than a broken-image icon, which at least announces itself. If the images cannot be supplied, deliver as normal and say plainly that the photo slots will be empty on that machine.

## Before you deliver: self-check

Re-read your own census against your own render. For each line in the census, find the matching value in the HTML. If any census line has no matching render output - a color, a count, a state - fix the render before sending it. A mismatch between the census and the render is the single failure mode this skill exists to prevent.

PHOTOS is the one line that walk can satisfy on a file whose photos never appear, because what it points at is the `<img>` tag and not the image. Check the tags themselves: every counted region is an `<img>` whose `src` is on `picsum.photos` or `i.pravatar.cc`, no two general photos share a `random` seed, and no region was quietly rendered as a gradient, a solid block or an SVG shape. Then say on delivery that those two hosts have to be reachable. It is the only census line whose render output lives somewhere other than the file.

Then walk color the other way, because the pass above cannot see this one. Search the file for `#` and for `rgb`, and account for every literal you find: it is inside the `:root` block, and it is either a color the census PALETTE line lists, a stop inside a gradient token whose role that line names, or a token carrying the not-from-the-reference comment from Step 3 rule 6. A hex that matches no census line still satisfies every census line, so census-to-render alone will pass a file that has quietly invented a color or hard-coded one in a rule.

Check the CUT OFF AT EDGE line at the census dimensions specifically: every item listed there is still clipped in the render, cut by the edge the census names, the cut falls inside the item rather than between two of them, and none has quietly grown into a whole one.

If either that line or SCROLL CUE is not `none`, confirm the frame exists before checking anything inside it: a container at the census width with `overflow` hidden rather than visible, and a height that is fixed only when the census names a cut at the bottom or top edge or a vertical scroll cue, `min-height` otherwise. A mockup that scrolls the page instead of the frame passes every count and still shows the clipped item in full, because the document grew to fit it. A height fixed on a horizontal cut fails the opposite way and is harder to catch: every count still matches, because every item is in the markup - the frame is hiding the ones past its edge.

Anything on the SCROLL CUE line needs the bar itself in the render, on the element and the edge the census names. `overflow-y: auto` is not that bar: with nothing clipped, the counted content does not overflow the frame, so `auto` renders nothing and this check fails on a file whose CSS reads correctly. `overflow-y: scroll` is not it either where the reference shows a thumb at a position, since nothing overflows there either and the track comes back full-length. Where the frame does hand the bar to the platform, check both axes are declared - `overflow-y: scroll` written alone puts `overflow-x` on `auto`, and the horizontal bar that can appear there is an element the reference never showed. Count the items while you are there - a render that overflows far enough to produce a bar of its own has rows the census never counted.

## Reference material

See [references/census-checklist.md](references/census-checklist.md) for the full census-field checklist and the state-fidelity rules in a single scannable list - read it when you want a compact pre-flight check before delivering a mockup, or when auditing someone else's mockup against a reference.
