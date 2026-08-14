# Changelog

## [1.2.0] - 2026-08-14

- Built the frame the clipping rule always assumed. Step 5 said to put a clipped item in "a container that overflows the frame", but nothing in the skill created a frame, and the default dimensions said the opposite: content height "may grow, never shrinks below 844". A document free to grow renders a half-visible row in full, which is the outcome Step 5 exists to forbid, so the two rules left no legal render.
- The census now decides whether the default height is a floor or a fixed edge. `CUT OFF AT EDGE: none` keeps the old behavior, since a full-page capture has no viewport edge and forcing one would invent a cut. Any entry on that line fixes the height, because that clipped item is the evidence that the reference's bottom edge is the viewport.
- Named the mechanism: one container at the census width and height with `overflow: hidden`, content in normal flow inside it, centered so the browser window size stops changing the mockup. `overflow-y: auto` only when the reference shows a scrollbar or the user asks for a scrollable prototype.
- Self-check and `references/census-checklist.md` now verify the frame before anything inside it - a page that scrolls instead of a frame that clips passes every count while showing the clipped item whole.
- The README example was the bug in miniature: its census listed a clipped row 4 and its body comment claimed the frame cut it, with no frame, no fixed height and no `overflow` anywhere in the file. It now builds one.

## [1.1.0] - 2026-08-08

- Added a `CUT OFF AT EDGE` census field: items the frame clips are now counted separately from the fully visible ones.
- Step 5 now separates clipped content from intentional empty space. A half-visible row is rendered clipped, never completed into a full row and never dropped - completing it invents content, dropping it turns a scrolling screen into a short one with dead space.
- New edge cases: device viewport vs an arbitrary crop (the cut only means "scrolls" in the first case), horizontally clipped carousels and wide tables, and a scrollbar with no partially visible item.
- Self-check and `references/census-checklist.md` extended to verify clipped items survive the render intact.

## [1.0.0] - 2026-07-12

- Initial release: census-first HTML mockup skill with the count-then-render method.
- Added `references/census-checklist.md` as a pre-flight/audit checklist for the census fields and state-fidelity rules.
- Documented default dimensions (mobile 390x844, tablet 768x1024, desktop 1440x900) and the photo-placeholder rule (real images, never gradient blobs).
- Added CI workflow validating `SKILL.md` frontmatter and the README link to it.
