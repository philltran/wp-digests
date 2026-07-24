# #80554: GradientPicker: select by slug so two presets sharing a gradient keep their identity

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`af54af7`](https://github.com/WordPress/gutenberg/commit/af54af73675368376a777144ec69a918c4d9d72c)
- **Discussion:** [#80554](https://github.com/WordPress/gutenberg/pull/80554) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `GradientPicker` component now supports slug-based selection via a new `selectedSlug` prop, fixing a bug where two gradient presets sharing the exact same gradient string would incorrectly render both as selected and commit the wrong preset slug when accepting an inherited value. The `onChange` callback also now passes the selected preset's slug as a third argument. This ensures that duplicate gradient presets defined in `theme.json` maintain their distinct identities in the block editor and Global Styles UI.

## Impact

- **Block editor & theme developers:** Fixes a data-integrity bug in the Global Styles gradient picker. If your `theme.json` defines multiple gradients with identical CSS values, the UI will now correctly track and commit the intended preset slug instead of defaulting to the first match.
- **Component consumers:** `GradientPicker` gains an optional `selectedSlug` prop. The `onChange` callback signature is extended to `(currentGradient, index, slug)`, but existing consumers are unaffected since the new arguments are additive.
- **No action required** for sites not using duplicate gradient presets or custom gradient pickers.

## Technical details

- `packages/components/src/gradient-picker/index.tsx`: `GradientPicker` now accepts `selectedSlug`. When provided, `isSelected` evaluates `slug === selectedSlug` instead of `value === gradient`. The `onChange` callback is updated to emit `(gradient, index, slug)`.
- `packages/block-editor/src/components/colors-gradients/control.js`: `ColorGradientControlInner` passes `gradientSlug` to `GradientPicker` and forwards the new `newSlug` argument to `onGradientChange`.
- `packages/block-editor/src/components/global-styles/background-panel.js` & `color-panel.js`: Extract preset slugs using `extractPresetSlug` and pass them as `inheritedSlug`/`userSlug` to the gradient control.
- `packages/block-editor/src/components/global-styles/hooks.js`: `encodeGradientValue` now accepts a `slug` parameter. If present, it returns `'var:preset|gradient|' + slug` directly, bypassing the string-matching logic that previously collapsed duplicates.
- `packages/components/src/gradient-picker/README.md`: Documents the new `selectedSlug` prop and updated `onChange` signature.

## Contribution

Opened by @ramonjd as a direct follow-up to #80497, which introduced slug-based selection for color palettes. The implementation mirrors the `ColorPalette` API to maintain consistency across the block editor's color and gradient controls. After review and manual testing by @noruzzamans and @andrewserong, the change was backported for WordPress 7.1 to align with the broader palette/gradient identity fixes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
