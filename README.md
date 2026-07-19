<div align="center">

<h1>HTML mockup</h1>

**Turns a reference screenshot into a pixel-faithful, single-file HTML mockup for designers and engineers who need a coded prototype that matches the reference exactly, not a generic reinterpretation.**

[![CI](https://img.shields.io/github/actions/workflow/status/humbleteam/html-mockup/validate.yml?branch=main&style=for-the-badge&logo=github&label=CI)](https://github.com/humbleteam/html-mockup/actions/workflows/validate.yml)
[![GitHub stars](https://img.shields.io/github/stars/humbleteam/html-mockup?style=for-the-badge&logo=github&color=181717)](https://github.com/humbleteam/html-mockup/stargazers)
[![Last commit](https://img.shields.io/github/last-commit/humbleteam/html-mockup?style=for-the-badge&color=339933)](https://github.com/humbleteam/html-mockup/commits/main)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-skill-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](https://github.com/humbleteam/html-mockup/blob/main/SKILL.md)

</div>

Paste a screenshot, ask for a coded version, and this Claude Code skill writes a census of the reference - exact palette, photo regions, item counts, component states - before touching a single line of HTML. The census becomes a contract: every value in it has to appear in the render, or the render is wrong. That one rule is what separates a mockup that matches a reference from a mockup that just resembles one.

## Table of contents

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Usage](#usage)
- [Example output](#example-output)
- [How it works](#how-it-works)
- [How is this different from just asking the model?](#how-is-this-different-from-just-asking-the-model)
- [FAQ](#faq)
- [Related skills](#related-skills)
- [Who maintains this](#who-maintains-this)

## What it does

- Writes a machine-checkable census (palette hex values, warm/cool/neutral temperature, photo regions, item counts per section, active states, button fills) as the first lines inside the HTML file, before any markup exists.
- Treats the census as a contract - every hex value, item count, and state description it lists must show up in the rendered mockup, or the render gets fixed before delivery.
- Replaces photo regions with real placeholder images (picsum.photos, i.pravatar.cc for faces), tinted toward the reference's color temperature with a CSS filter - never a gradient standing in for a photograph.
- Locks color to hex values extracted from the reference, declared once as CSS custom properties on `:root`, with no raw hex anywhere else in the stylesheet.
- Reproduces component states exactly - filled vs outlined buttons, two-signal active tabs, elevated floating action buttons, correctly sized notification dots.
- Renders the exact number of items counted in the reference, never padding a section to fill the viewport.

## Quick start

Install as a personal skill, available in every project:

```bash
git clone https://github.com/humbleteam/html-mockup ~/.claude/skills/html-mockup
```

Install as a project skill, checked into one repo:

```bash
git clone https://github.com/humbleteam/html-mockup .claude/skills/html-mockup
```

Using a different agent (Cursor, Codex, or any LLM-based agent): the skill is plain markdown in the [Agent Skills](https://github.com/humbleteam/html-mockup/blob/main/SKILL.md) format. Paste `SKILL.md` into the system prompt or context window.

To verify the install in Claude Code, restart Claude Code and check that the skill is listed - skills load from `~/.claude/skills/` (personal) and `.claude/skills/` (project).

## Usage

- "Here's a screenshot of our onboarding screen - build an HTML mockup that matches it exactly." The skill writes the census comment first, then renders one self-contained HTML file against it.
- "Turn this Dribbble shot into a working mobile mockup at 390x844." It extracts the palette and counts every item before writing any CSS.
- "This mockup doesn't match the reference - the buttons look wrong." It re-reads the reference's button fills and corrects the specific state mismatch instead of regenerating the whole file.

## Example output

The first lines inside the delivered `.html` file always look like this, before any `<!doctype html>`:

```html
<!-- CENSUS:
  PALETTE: bg=#F5F2EB text=#1A1A1A accent=#C9762F secondary=#7A6A52
  TEMPERATURE: warm
  PHOTOS: 4 regions - hero portrait 390x480, avatar circle 48x48 x3
  ITEMS PER SECTION: 3 list rows, 5 nav tabs
  ACTIVE STATES: Home tab: filled icon + accent-color dot indicator
  BUTTON FILLS: Save: filled #1A1A1A, Cancel: outlined #7A6A52
-->
<!doctype html>
<html lang="en">
<head>
<meta charset="UTF-8">
<style>
  :root {
    --bg-primary: #F5F2EB;
    --text-primary: #1A1A1A;
    --accent: #C9762F;
    --secondary: #7A6A52;
  }
  body { background: var(--bg-primary); color: var(--text-primary); }
  .avatar { border-radius: 50%; width: 48px; height: 48px; object-fit: cover; }
</style>
</head>
<body>
  <img class="avatar" src="https://i.pravatar.cc/96?img=12" alt="User avatar">
  <!-- exactly 3 list rows, matching ITEMS PER SECTION -->
</body>
</html>
```

(Example fixture only - the palette and counts above are illustrative, not from a real client project.)

## How it works

- **Census before code.** The reference gets studied and counted first - palette, temperature, photo regions, item counts, active states, button fills - as a comment block that is the literal first thing in the output file.
- **The census is the contract.** Every field written in it must have a matching value in the render. A mismatch is a bug in the render, not a detail to shrug off.
- **Photos are photos.** Any photographic region gets a real `<img>` from a placeholder photo service, tinted toward the reference's warmth with a CSS filter. Gradients simulating photographs are disallowed.
- **Color is extracted, not approximated.** Hex values come from close observation, declared once as named CSS custom properties on `:root`, used nowhere else as raw hex - which also keeps warm/cool temperature consistent across every neutral.
- **States get rendered as-is.** Filled buttons stay filled, outlined stay outlined, and an active tab needs at least two simultaneous visual signals to read as selected - a color swap alone isn't enough.
- **Counts are locked.** Three list rows in the reference means three in the output. Nothing gets added to fill space, nothing gets dropped to save it.
- **Icons are inline SVG with a matching metaphor**, never emoji or a generic stand-in for the shape actually shown in the reference.
- **Defaults fill the gaps.** Mobile 390x844, tablet 768x1024, desktop 1440x900 when the device isn't specified; nearest system font stack, flagged in a comment, when the typeface can't be identified.

## How is this different from just asking the model?

A bare "turn this screenshot into HTML" prompt tends to reinterpret the reference loosely: a plausible item count instead of the actual one, a gradient standing in for a photo, every button defaulted to the same filled style, and rarely one exact hex value held consistently across the file. The cause is a missing forcing function. This skill adds one: a mandatory pre-render census that becomes a literal checklist, plus fixed rules for photos, color, and component states, so the output converges on the specific reference instead of a generic mockup that merely looks similar.

## FAQ

**How do I get an AI to match a design exactly?**
Force it to count and name everything in the reference before it writes any code - exact colors, item counts, component states - then check the output against that list. That's the method this skill automates.

**Why do AI mockups look generic?**
Usually the model renders its own idea of "a settings screen" instead of the specific one in the reference: a plausible item count, a default button style, a gradient instead of a photo. Pinning down exact counts and states before rendering removes most of that drift.

**Can Claude convert a screenshot to HTML?**
Yes, when it can see the image (pasted into chat or a URL it can view) and is given an explicit method for extracting colors, counting items, and preserving component states - which is what this skill provides. Without that method, the output tends to drift toward a generic reinterpretation of the layout instead of a match.

**What is count-then-render?**
The two-step discipline this skill runs on: write down everything counted in the reference - colors, photos, item counts, states - then render exactly that, nothing added, nothing dropped. The count happens before any HTML is written, so it doubles as a checklist for verifying the output afterward.

**Does this work without a reference image?**
It works best with one. Without an image, the skill falls back to a described layout but still writes the census first, with every guessed field marked as an assumption to correct.

**What image formats work as a reference?**
Any screenshot Claude can view in the conversation - PNG, JPG, or a pasted image - works. A live URL works too if the agent can render or fetch it.

**What does the output actually contain?**
One self-contained `.html` file: full document structure, an inline `<style>` block with CSS custom properties, no external stylesheet, framework, or build step. It opens directly in a browser.

## Related skills

Part of a 10-skill open-source kit for design teams by Humbleteam.

- [design-review](https://github.com/humbleteam/design-review) - structured UX critique with a 0-4 score, Before/After/Why fixes, and a citation for every claim.
- [ascii-wireframes](https://github.com/humbleteam/ascii-wireframes) - three distinct layout hypotheses as ASCII wireframes before any hi-fi work.
- [extract-design-tokens](https://github.com/humbleteam/extract-design-tokens) - pull palette, type, spacing, radii, and shadows from a URL or screenshot into CSS variables and JSON.
- [audit-design-tokens](https://github.com/humbleteam/audit-design-tokens) - find token drift in a codebase: raw hex values, off-scale spacing, near-duplicate colors.
- [design-qa](https://github.com/humbleteam/design-qa) - a pre-ship design QA gate: states, contrast, touch targets, breakpoints, keyboard paths.
- [design-handoff](https://github.com/humbleteam/design-handoff) - turn a finished mockup into a dev-ready spec: tokens, states, accessibility annotations, open questions.
- [accessibility-audit](https://github.com/humbleteam/accessibility-audit) - WCAG 2.2-grounded accessibility review with success-criterion citations and severity levels.
- [ux-writing](https://github.com/humbleteam/ux-writing) - interface copy that reads human: plain-verb microcopy rules and an AI-tell strip pass.
- [design-brief](https://github.com/humbleteam/design-brief) - extract a 5-bullet design brief from messy project inputs, with a gap report for what is missing.

## Who maintains this

Maintained by [Humbleteam](https://humbleteam.com/ai), a design and AI-engineering studio that builds AI infrastructure for design teams. This skill is distilled from the internal playbooks we run on client work. Issues and PRs welcome.

MIT - see [LICENSE](LICENSE).
