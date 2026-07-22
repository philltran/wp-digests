# #78795: Add support for aspect ratio and related controls in viewport states

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Block editor`, `[Package] Style Engine`, `[Feature] Style States`
- **Merged:** [`0b846fc`](https://github.com/WordPress/gutenberg/commit/0b846fc777bb98f85b413da7998b152e22c7effe)
- **Discussion:** [#78795](https://github.com/WordPress/gutenberg/pull/78795) · 24 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This PR enables `aspectRatio`, `width`, `height`, and `minHeight` controls to function correctly within viewport states for the Image, Featured Image, and Cover blocks. Previously, setting these dimensions in a viewport state would not apply responsively or would conflict with default CSS values. The change introduces explicit validation for aspect ratio values and automatic fallback logic to unset conflicting dimension properties per viewport state, ensuring responsive dimension controls work as intended in the block editor.

## Impact

- **Block & theme developers:** The `DimensionsTool` component now defaults to `cover` instead of `fill` for the scale control. Custom implementations that rely on the previous default or manually override `defaultScale` may need adjustment.
- **Site owners / editors:** Aspect ratio and height controls now apply correctly when configured per viewport state. Setting an aspect ratio automatically unsets `height`/`min-height` for that state, and vice versa.
- **No action required** for existing sites unless they depend on the `DimensionsTool` default scale value or custom viewport state dimension logic.

## Technical details

- Added `gutenberg_is_explicit_aspect_ratio_value()` in `lib/block-supports/dimensions.php` to distinguish explicit ratios from `'auto'` or empty strings, preventing unintended CSS overrides.
- Updated `gutenberg_render_dimensions_support()` and `gutenberg_get_block_state_style_rules()` to apply fallback styles that unset conflicting properties (`height`/`minHeight` vs `aspectRatio`) when rendering block and state styles.
- Introduced `isExplicitAspectRatio()` in `packages/block-editor/src/hooks/dimensions.js` and `getStateFallbackDimensionStyles()` in `packages/block-editor/src/hooks/style.js` to handle client-side `useBlockProps` class assignment and state CSS compilation.
- Changed `DimensionsTool` default scale from `'fill'` to `'cover'` in `packages/block-editor/src/components/dimensions-tool/index.js` and updated `scale-tool.js` to respect the passed `defaultValue` prop.
- Updated `hasAspectRatio()` in `packages/block-editor/src/components/global-styles/dimensions-panel.js` to use `!hasPseudoBlockStyleState()` instead of `isDefaultBlockStyleState()`.
- Usage pattern change in `useBlockProps`:
  ```js
  // Before
  'has-aspect-ratio': !! style?.dimensions?.aspectRatio,
  // After
  'has-aspect-ratio': isExplicitAspectRatio( style?.dimensions?.aspectRatio ),
  ```

## Contribution

Opened by @tellthemachines as part of #77817, the PR was reviewed and merged with co-authors @andrewserong, @ramonjd, and @talldan. During review, the author noted an initial consideration to standardize these dimension controls as formal block supports, but the team agreed to ship the per-block implementation first to avoid scope creep, with block support standardization slated for a follow-up. The diff reflects the finalized per-block tool integration without the broader block-support refactor.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
