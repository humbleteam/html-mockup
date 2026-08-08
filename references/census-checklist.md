# Census checklist

Read this before delivering a mockup, as a compact pre-flight check against the reference. It restates the census fields and the state-fidelity rules from `SKILL.md` in checklist form so you can walk through them fast, or use it to audit a mockup someone else produced.

## Census fields

- [ ] **PALETTE** - every distinct color in the reference is listed with an exact hex value, not a nearest-named color. Backgrounds, text, accent, and secondary colors are all present.
- [ ] **TEMPERATURE** - the reference is classified warm, cool, or neutral, and every neutral color in the render matches that temperature.
- [ ] **PHOTOS** - every photographic region is counted with its approximate dimensions and content ("hero portrait 390x500", "food circle 60x60 x3").
- [ ] **ITEMS PER SECTION** - every repeating group (list rows, cards, tabs, buttons) has an exact count, not an approximation.
- [ ] **CUT OFF AT EDGE** - every item the frame clips is listed with how much of it shows ("list row 4, top half visible"), counted separately from the fully visible ones, or the line reads `none`.
- [ ] **ACTIVE STATES** - the selected/active element in each group is named, with the specific visual signal that marks it as active.
- [ ] **BUTTON FILLS** - every button is classified filled-dark, filled-color, outlined, or ghost, with its exact fill or border color.

## State-fidelity rules

- [ ] Filled buttons use `background-color`, not a border-only style, when the reference shows a solid fill.
- [ ] Outlined buttons use `border` with a transparent background when the reference shows no fill.
- [ ] The active navigation tab differs from inactive tabs by at least two simultaneous signals (icon style, color, indicator dot/line, weight) - a single color change is not enough.
- [ ] Floating action buttons have visible elevation (`box-shadow`), correct size (56-64px), and correct overlap with whatever they sit above.
- [ ] Notification dots are sized 8-10px, colored to match the reference, and have a ring in the surrounding background color.
- [ ] Photo regions use real `<img>` tags (picsum.photos, i.pravatar.cc), never a gradient.
- [ ] No raw hex values exist outside the `:root` custom-property block.
- [ ] Item counts in the render match item counts in the census exactly - no added "filler" items, nothing dropped to save space.
- [ ] Every clipped item is still clipped in the render, cut inside the item rather than in the gap between two - never completed into a full row, never dropped, never replaced by empty space.
- [ ] Icons are inline SVG with a metaphor that matches the label, never emoji.
- [ ] Any UI gradient (not a photo) has at least 3 color stops.

## Final pass

Walk the census top to bottom one more time. For each line, point to the exact place in the rendered HTML that satisfies it. A census line with nothing to point to is a bug - fix the render, not the checklist.
