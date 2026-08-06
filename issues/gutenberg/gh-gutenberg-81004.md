# #81004: [Backport to 7.1] Add opt-out for block style state controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`a5fd6ad`](https://github.com/WordPress/gutenberg/commit/a5fd6ad86f84733f8be175d01051411b00b24f55)
- **Discussion:** [#81004](https://github.com/WordPress/gutenberg/pull/81004) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This change introduces a new `blockStatesEnabled` editor setting that allows developers to hide block style state controls in the Block Inspector and Global Styles UI. When set to `false`, the UI elements for applying pseudo-state styles are removed, though any state styles already saved in `theme.json` or block attributes continue to render normally. This gives theme and plugin authors finer control over the editor experience by letting them disable state editing for specific sites or user roles.

## Impact

- **Plugin & theme developers**: Can now conditionally disable block state editing via the `block_editor_settings_all` filter.
- **Site owners & editors**: Will see fewer controls in the inspector and Global Styles if the setting is disabled.
- **No breaking changes**: The setting defaults to `true`, preserving existing behavior. Existing state styles remain unaffected.
- **Action required**: None, unless you want to hide state controls.

## Technical details

The diff adds `blockStatesEnabled` to `SETTINGS_DEFAULTS` in `packages/block-editor/src/store/defaults.js` and `EDITOR_SETTINGS_DEFAULTS` in `packages/editor/src/store/defaults.js`, defaulting to `true`. It is registered in `BLOCK_EDITOR_SETTINGS` in `packages/editor/src/components/provider/use-block-editor-settings.js`. In `packages/block-editor/src/components/block-inspector/index.js`, the `BlockInspectorSingleBlock` component conditionally renders `<BlockStatesControl>` based on the `blockStatesEnabled` prop. In `packages/global-styles-ui/src/screen-block.tsx`, the `ScreenBlock` component passes `pseudoStates={ showBlockStateControls ? validPseudoStates : [] }` to the state picker. The setting is exposed via `block_editor_settings_all` and documented in `docs/reference-guides/filters/editor-filters.md`.

## Contribution

Opened by @andrewserong as a manual backport of #80956 to the `wp/7.1` branch to resolve changelog merge conflicts. Co-authored by @talldan and @ramonjd. The PR received minimal review discussion, with the primary focus on cleanly applying the original feature to the release branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
