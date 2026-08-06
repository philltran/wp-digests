# #80814: Add a responsiveEditingEnabled editor setting to hide the Responsive styles option

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Feature] Extensibility`, `[Package] Editor`, `has dev note`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`e438f35`](https://github.com/WordPress/gutenberg/commit/e438f3552f5c395aa5b7f48941f3c7c8a5e77ba4)
- **Discussion:** [#80814](https://github.com/WordPress/gutenberg/pull/80814) · 7 comments · 3 reactions
- **Usefulness:** 4/5

## Summary

Adds a `responsiveEditingEnabled` editor setting that allows site administrators to hide the "Responsive styles" toggle in the View menu and the viewport state control in Global Styles. Introduced in response to feedback from the responsive styling call for testing, this setting gives teams a way to opt out of per-viewport style editing in the block editor without removing existing responsive styles from content or the front end.

## Impact

- **Plugin & theme developers**: Can restrict editors from creating viewport-specific styles by filtering `block_editor_settings_all`.
- **Site owners & editors**: Gain a configuration option to simplify the editor UI if responsive styling isn't part of their workflow.
- **Breaking changes / deprecations**: None. The setting defaults to `true`, preserving existing behavior.
- **Migration / configuration**: No action required. Add the filter only if you want to disable the feature.
- **Front-end rendering**: Completely unaffected. Styles already saved in `theme.json`, Global Styles, or block attributes continue to output normally.

## Technical details

The change introduces `responsiveEditingEnabled` to the `EDITOR_SETTINGS_DEFAULTS` object in `packages/editor/src/store/defaults.js`, defaulting to `true`. The `PreviewDropdown` component in `packages/editor/src/components/preview-dropdown/index.js` now reads the setting via `getEditorSettings()` and conditionally renders the "Responsive styles" `MenuItem` using `{ isResponsiveEditingEnabled && ( <MenuGroup> ... ) }`. In `packages/editor/src/components/global-styles/index.js`, the `GlobalStylesUIWrapper` component merges the new setting with the existing `showResponsiveStateControls` prop (`showResponsiveStateControls={ showResponsiveStateControls && responsiveEditingEnabled }`), effectively hiding the viewport state control in Global Styles when disabled. The setting is exposed to PHP via the `block_editor_settings_all` filter, following the established pattern for `richEditingEnabled` and `codeEditingEnabled`.

## Contribution

Opened by @t-hamano following feedback from the responsive styling call for testing and a Slack discussion. The implementation was reviewed and refined by @andrewserong, who suggested a wording tweak to the documentation to clarify that the setting gates editing behavior rather than front-end output. @ramonjd was also credited as a co-author. The PR was merged and subsequently backported to the `wp/7.1` branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
