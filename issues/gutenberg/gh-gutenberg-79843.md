# #79843: Quote block: add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Quote`
- **Merged:** [`0a73d76`](https://github.com/WordPress/gutenberg/commit/0a73d765115b269debd807a241b11a7d1cc374ea)
- **Discussion:** [#79843](https://github.com/WordPress/gutenberg/pull/79843) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Quote block now routes background gradients through the modern `background` support path instead of the legacy `color.gradients` path. This resolves a CSS conflict where applying both a gradient and a background image would cause the image declaration to override the gradient. By enabling `supports.background.gradient`, the editor now outputs a single combined `background-image` declaration containing both values.

## Impact

- **Plugin & theme developers:** No immediate code changes required. Existing Quote blocks will continue to render, but the underlying CSS output structure changes.
- **Site owners & editors:** The gradient control remains visible by default. When both a gradient and background image are applied, they now render correctly together instead of the image hiding the gradient.
- **No action required** for existing sites or custom themes, though developers who manually parsed or targeted the legacy `background: linear-gradient(...)` output pattern will need to update their CSS selectors.

## Technical details

The change modifies `packages/block-library/src/quote/block.json` to add `"gradient": true` under `supports.background` and includes it in `background.__experimentalDefaultControls` to preserve UI visibility. This routes gradient styles through the block editor’s background support system, serializing them to `style.background.gradient` alongside `style.background.backgroundImage`. The CSS output shifts from conflicting declarations:
```css
/* Before */
background: linear-gradient(...);
background-image: url(...);
```
to a single combined declaration:
```css
/* After */
background-image: linear-gradient(...), url(...);
```
Block documentation in `packages/block-library/src/quote/README.md` and `docs/reference-guides/core-blocks/README.md` is updated to reflect the new `background.gradient` support flag.

## Contribution

Opened and merged by @ramonjd as a direct follow-up to PR #75859. The implementation focused on aligning the Quote block’s gradient handling with the established background support path to fix the CSS override bug. No significant design debates or alternative approaches were documented in the PR discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
