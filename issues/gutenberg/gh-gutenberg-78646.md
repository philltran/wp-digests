# #78646: Button: Align focus styles with design system

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`e36e635`](https://github.com/WordPress/gutenberg/commit/e36e635898e728c19a78dc222d41a3fdab0efd4a)
- **Discussion:** [#78646](https://github.com/WordPress/gutenberg/pull/78646) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/components` Button styles now replace variant-specific box-shadow focus indicators with a shared `outset-ring__focus` mixin across all states. This aligns the legacy component's keyboard focus behavior with the modern design system and `@wordpress/ui`, ensuring consistent visual feedback during navigation. The update also restructures high-contrast mode boundaries into dedicated media queries to eliminate outline flicker and layout shifts.

## Impact

- **Plugin & Theme Developers (editor/Gutenberg consumers):** Visual focus indicators for the `<Button />` component will render as an outlined ring instead of a box-shadow. Custom CSS that previously targeted `.components-button:focus` or variant-specific `box-shadow` rules may need adjustment to avoid unintended overlapping outlines in app surfaces.
- **Site Owners / Editors:** Keyboard navigation focus states will be visually consistent across primary, secondary, tertiary, destructive, and link variants. No configuration changes required.
- **No action required** for standard implementations of the Button component, though custom overrides targeting focus state borders or box-shadows should be audited to match the new mixin behavior.

## Technical details

The change modifies `packages/components/src/button/style.scss` (and logs it in the components CHANGELOG). Key diff-driven updates:
- Imports `@wordpress/base-styles/mixins` and applies `@include mixins.outset-ring__focus();` to `.components-button:focus:not(:active)` at the component root.
- Removes the global `transition: box-shadow 0.1s linear;` from the base `.components-button` rule and moves it exclusively to the `.is-secondary` variant (which previously relied on inset shadows for its border).
- Relocates forced-colors/high-contrast boundaries (`outline: 1px solid ButtonBorder`) into scoped `@media (forced-colors: active)` blocks, replacing older transparent-outline workarounds that caused flicker.
- Strips the destructive variant's custom red focus ring, standardizing it to the component accent color to align with `@wordpress/ui`.
- Removes specificity-heavy selectors like `&:focus:active { outline: none; }` and disabled state focus overrides that were interfering with downstream component styling.

## Contribution

Opened by @mirka and co-authored with @ciapo and @jasmussen, the PR was merged after confirming it successfully unified focus rendering across the Button matrix. During testing, reviewers noted that the new global focus outline exposed unintended visual overlaps in several app-surface overrides (e.g., Posts button, Block Inserter, Document Bar), documenting these as known follow-ups rather than blocking the merge. The team prioritized design system alignment and accessibility consistency over incremental custom surface adjustments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
