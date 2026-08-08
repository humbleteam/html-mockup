# Changelog

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
