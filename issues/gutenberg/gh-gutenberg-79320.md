# #79320: Icons: self declare color

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Icons`, `has dev note`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`c5cdf2a`](https://github.com/WordPress/gutenberg/commit/c5cdf2a8532283022530eb4a3c70725d30b63a5a)
- **Discussion:** [#79320](https://github.com/WordPress/gutenberg/pull/79320) · 5 comments · 0 reactions
- **Usefulness:** 5/5

## Summary

The `@wordpress/icons` package now injects `fill="currentColor"` directly into the outer `<svg>` element of all 330 shipped icons. This ensures every icon inherits the CSS `color` property from its parent by default, making color customization more reliable across icons that use `fill`, `stroke`, or mixed attributes. The change is a documented breaking shift for developers who previously relied on CSS `fill` rules to tint icons.

## Impact

- **Plugin & theme developers:** If you style `@wordpress/icons` SVGs using CSS `fill`, those rules will no longer override the icon's color. Switch to setting `color` on the wrapper element instead.
- **Block & component authors:** Custom or third-party block icons rendered via `BlockIcon` will now follow inherited `color` rather than ancestor `fill` rules. Existing wrapper styles targeting `.block-editor-block-icon` remain safe, but explicit `fill` overrides on icon wrappers will be ignored.
- **No action required** for users relying on default icon colors or using the `color` CSS property for theming.

## Technical details

The PR modifies the SVG source files in `packages/icons/` by appending `fill="currentColor"` to the root `<svg>` tag. The transformation replaces:
```diff
-<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
+<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
```
A new helper function `normalizeSourceIcons()` was added to `packages/icons/lib/generate-library.cjs` to automate this transformation during library generation. The script intentionally skips SVGs that already declare a `fill` attribute or use `style="fill: none"` (common in stroke-based icons). This change affects the compiled output of `@wordpress/icons` (version 15.0.0) and propagates to any component consuming it, including `BlockIcon` and `DataViews` icon slots.

## Contribution

Opened as a follow-up to review feedback on #78808, the PR was co-authored with Claude Opus 4.8 to handle the mechanical but large-scale update across 330 icons. During review, @mirka verified that existing wrapper styles in `block-manager/style.scss` and `BlockIcon` would not cause regressions, noting that inheriting `color` aligns with the expected UI text color while preserving flexibility for custom or third-party icons. The team opted to ship the normalization as a standalone PR for easier review, with the `normalizeSourceIcons` helper left as a manual maintenance tool for future icon additions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
