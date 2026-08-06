# #80956: Add opt-out for block style state controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block editor`, `Needs Dev Note`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`bdaa119`](https://github.com/WordPress/gutenberg/commit/bdaa1192dc62bdabd229ac237c309f37a74f0874)
- **Discussion:** [#80956](https://github.com/WordPress/gutenberg/pull/80956) · 15 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Introduces a `blockStatesEnabled` editor setting that allows developers to hide the block style state controls in the block inspector and Global Styles UI. This opt-out complements the existing `responsiveEditingEnabled` setting, giving site owners and theme developers complete control over whether users can edit pseudo-state styles (like `:hover` or `:focus`) through the editor interface.

## Impact

- **Theme & plugin developers:** Can now disable state editing entirely by setting `blockStatesEnabled` to `false` via the `block_editor_settings_all` filter.
- **Site owners / editors:** Will see no immediate change unless a theme or plugin explicitly disables the feature.
- **No breaking changes:** Existing state styles saved in `theme.json`, Global Styles, or block attributes remain fully functional on the front end and in the editor.
- **No migration required:** The setting defaults to `true`, preserving current behavior.

## Technical details

The diff adds `blockStatesEnabled` (defaulting to `true`) to the editor settings schema in `packages/block-editor/src/store/defaults.js` and `packages/editor/src/store/defaults.js`. It is passed through `packages/editor/src/components/provider/use-block-editor-settings.js` and consumed in `packages/block-editor/src/components/block-inspector/index.js` to conditionally render the `BlockStatesControl`. In the Global Styles UI, the setting is read in `packages/editor/src/components/global-styles/index.js` and passed as `showBlockStateControls` to `GlobalStylesUI` in `packages/global-styles-ui/src/global-styles-ui.tsx`. The actual UI toggle is handled in `packages/global-styles-ui/src/screen-block.tsx`, where `pseudoStates` is set to an empty array when `showBlockStateControls` is `false`. The change is documented in `docs/how-to-guides/curating-the-editor-experience/disable-editor-functionality.md` and `docs/reference-guides/filters/editor-filters.md`.

## Contribution

Opened by @talldan to address issue #80850, the PR was initially met with hesitation by @andrewserong, who noted that adding granular config flags could become a "slippery slope." After review, @andrewserong and @ramonjd validated the approach, noting it aligns with existing patterns for hiding block support controls. @ramonjd performed manual testing across all combinations of `blockStatesEnabled` and `responsiveEditingEnabled`. The change was merged to trunk and subsequently backported to the `wp/7.1` branch via #81004.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
