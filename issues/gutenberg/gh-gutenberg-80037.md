# #80037: Enable text alignment to be set by viewport state

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`5db1249`](https://github.com/WordPress/gutenberg/commit/5db12497143ed78e0f8a2a4271eabf25f22d3f68)
- **Discussion:** [#80037](https://github.com/WordPress/gutenberg/pull/80037) · 4 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

This PR enables the block editor’s text alignment control to be set per viewport state (e.g., `@mobile`, `@tablet`). Previously, responsive editing hid the text alignment control because it relied on class-based serialization, which conflicted with the new state-based style system. The change routes the control to a dedicated `style-state` toolbar slot and generates `text-align` as an inline `!important` CSS declaration scoped to viewport media queries, ensuring it correctly overrides the default class-based alignment.

## Impact

- **Block & theme developers:** No immediate code changes required. The change is internal to the block editor and core rendering pipeline.
- **Site owners / editors:** Enables setting text alignment independently for mobile/tablet breakpoints in the responsive editor without losing the default desktop alignment.
- **Hosting & platform teams:** No action required. The change ships in Gutenberg and is backported to WordPress 7.1.
- **Note:** If you previously worked around the missing alignment control in responsive mode, you can remove those workarounds.

## Technical details

- **PHP (`lib/block-supports/states.php`):** `gutenberg_add_block_state_style_rule()` now reads `$style['typography']['textAlign']` and injects it into `$declarations` as a CSS property instead of relying on class names. This ensures state styles output `text-align: right !important;` inside media queries rather than appending a class.
- **JS (`packages/block-editor/src/hooks/text-align.js`):** `BlockEditTextAlignmentToolbarControlsPure` now uses `useSelect` to fetch `selectedState` and `isResponsiveEditing` from `blockEditorStore`. It routes the `AlignmentControl` to the `style-state` `BlockControls` group when a viewport state is active, and uses `getStyleForState()` / `setStyleForState()` to read/write alignment per state.
- **JS (`packages/block-editor/src/hooks/style.js`):** Adds `getStateTextAlignCSS()` to generate `text-align: <value> !important` declarations for state styles, integrated into `getStateStylesCSS()`.
- **JS (`packages/block-editor/src/components/block-controls/groups.js` & `block-toolbar/index.js`):** Introduces a new `BlockControlsStyleState` slot/fill for the `style-state` group. The toolbar conditionally renders this slot when `_isEditingResponsiveStyleState` is true.
- **JS (`packages/block-editor/src/hooks/layout.js`, `constrained.js`, `flex.js`):** Layout controls (justification, vertical alignment) now accept a `controlsGroup` prop, defaulting to `block` but switching to `style-state` during responsive viewport editing.
- **Tests:** Added PHPUnit test `test_responsive_text_alignment_generates_media_query_scoped_css()` and JS tests for `getStateTextAlignCSS` and `getTextAlignControlGroup`.

## Contribution

Opened by @tellthemachines and merged after review by @talldan and @andrewserong. The PR addresses a gap identified in #80002 where responsive editing hid the text alignment control. The author noted that text alignment differs from other typography tools because it serializes as class names, requiring separate handling in state processing. The implementation introduces a dedicated `style-state` toolbar slot and routes alignment/layout controls there, with tests validating both PHP rendering and JS state management.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
