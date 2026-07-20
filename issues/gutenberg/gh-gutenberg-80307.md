# #80307: Responsive styles: Open inspector sidebar when toggling

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`6eb60dd`](https://github.com/WordPress/gutenberg/commit/6eb60dd879245cba7db80acd29ef0c79657c520a)
- **Discussion:** [#80307](https://github.com/WordPress/gutenberg/pull/80307) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

When a user enables responsive editing in the block editor, the block inspector sidebar now automatically opens if a block is currently selected. This change addresses a discoverability gap where users might toggle responsive styles without realizing the styling controls are located in the inspector. It streamlines the responsive editing workflow by surfacing relevant UI immediately after activation.

## Impact

- **Block editor users:** The inspector sidebar will automatically open when enabling responsive editing on a selected block, reducing friction in the responsive styling workflow.
- **Plugin & theme developers:** No direct API changes, deprecations, or breaking changes. The behavior is internal to the editor UI.
- **Hosting & platform teams:** No configuration, migration, or code changes required.
- **Headless & REST consumers:** Unaffected.

## Technical details

The change modifies `packages/editor/src/components/preview-dropdown/index.js` in the Gutenberg package. It imports `enableComplementaryArea` from `@wordpress/interface` and `sidebars` from `../sidebar/constants`. The `useSelect` hook now tracks `hasBlockSelection` by calling `getBlockSelectionStart()` from `blockEditorStore`. Inside the `handleDevicePreviewChange` callback, when `newIsResponsiveEditing` becomes true and `hasBlockSelection` is true, the component calls `enableComplementaryArea( 'core', sidebars.block )` to open the block inspector. This leverages the existing `interfaceStore` complementary area API rather than introducing new hooks or filters.

## Contribution

Opened by @talldan as an experimental UX improvement to guide users after toggling responsive editing. Reviewed and tested by @tellthemachines, @andrewserong, @noruzzamans, and @ramonjd. The author noted uncertainty about the sidebar opening "underneath the dropdown," but the change was approved and merged. It was cherry-picked to `wp/7.1` for inclusion in the next release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
