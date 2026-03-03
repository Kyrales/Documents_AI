# Design

## CSS Approach

### Buttons
Target `.action` and `button` elements.
- Background: `var(--sl-color-black)`
- Text: `var(--sl-color-white)` or `var(--sl-color-accent-low)` (neon).
- Hover: Inverse or offset shadow.

### Social Icons
Starlight uses SVGs for social icons.
- Selector: `a.social-icons svg` (need to verify exact selector).
- Fix: Set `fill: currentColor` or explicit black/neon.

### Draft Badge
- Class: `.badge-draft`
- Style: Yellow background (`#ffeb3b`), black text, 2px border.

## Content Structure

- Create `src/content/docs/methodology/`
- Copy source MD files to this directory.
- Rename files to kebab-case for better URLs:
    - `SDD_OpenSpec.md` -> `sdd-openspec.md`
    - `SDD_SpecKit.md` -> `sdd-speckit.md`
    - `SDD_Template.md` -> `sdd-template.md`

## Configuration

Update `astro.config.mjs`:
- Add `{ label: 'Methodology', autogenerate: { directory: 'methodology' } }` to `sidebar` array.
