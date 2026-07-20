# #80289: Hide color controls for Navigation and Social Icons when viewport sta…

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block library`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`f909ebf`](https://github.com/WordPress/gutenberg/commit/f909ebf9606344ad8a8d26c10de07996add5dfb5)
- **Discussion:** [#80289](https://github.com/WordPress/gutenberg/pull/80289) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Navigation and Social Icons blocks now hide their color panel controls in the block editor when a responsive viewport state (Tablet or Mobile) is active. This prevents users from interacting with color settings that do not support viewport-specific styling, as these blocks rely on custom color implementations rather than standard block color supports. The change aligns the editor UI with the current limitations of the Style States feature.

## Impact

- **Block & theme developers**: No code changes or API updates required. The behavior is strictly editor-side UI logic.
- **Site owners & editors**: Color controls will temporarily disappear when editing Navigation or Social Icons blocks in Tablet/Mobile viewports. Other responsive controls (Background, Layout, Dimensions, Border) remain fully functional.
- **No breaking changes or migration steps**.

## Technical details

- Modified `packages/block-library/src/navigation/edit/index.js` and `packages/block-library/src/social-links/edit.js`.
- Both files now access the internal `hasSelectedStyleState` selector via `unlock( select( blockEditorStore ) ).hasSelectedStyleState( clientId )`.
- In `navigation/edit/index.js`, the `<InspectorControls group="color">` wrapper is now gated by `{ ! hasSelectedStyleState && (...) }`. The `useSelect` dependency array was also updated from `[]` to `[ clientId ]` to ensure reactivity.
- In `social-links/edit.js`, a `showColorControls` flag is computed as `colorGradientSettings.hasColorsOrGradients && ! hasSelectedStyleState`, which conditionally renders the color panel.
- This leverages the internal Style States selector to detect active viewport states, ensuring custom color tools (`<ColorTools>` in Navigation, `colorSettings` mapping in Social Links) are hidden during responsive editing.

## Contribution

Opened by @tellthemachines as part of the Style States initiative (#77817). The PR was reviewed and merged with co-authors @andrewserong and @noruzzamans. The author noted that migrating to default color block support or enabling viewport-specific styles for custom controls were considered alternatives, but hiding the controls was chosen for the 7.1 beta cycle. It was quickly backported to the Gutenberg RC and WordPress 7.1 release branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
