# #79417: Pattern editing: show root block identity when editing pattern sections

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Patterns`
- **Merged:** [`5e2c29b`](https://github.com/WordPress/gutenberg/commit/5e2c29b9f2061372c2904e4bffb2f0af494311f4)
- **Discussion:** [#79417](https://github.com/WordPress/gutenberg/pull/79417) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

When editing a pattern section or viewing a template part in an isolated editor, the block editor now displays the underlying root block’s identity instead of the generic pattern symbol. This clarifies which wrapper block (e.g., Group, Cover) is actually being modified, improving context awareness during pattern editing. The change ensures the toolbar, list view, and inspector consistently reflect the active editing context rather than forcing pattern metadata.

## Impact

- **Site editors & theme developers:** No code changes required. The UI behavior changes only when actively editing pattern sections or using isolated template part editors.
- **Plugin & theme developers:** No breaking changes to public APIs. If you rely on internal block editor components for custom UI that assumes pattern metadata is always shown, you may need to account for the new `isWithinEditedContentOnlySection` and `isIsolatedEditor` conditions.
- **No action required** for standard site or plugin development.

## Technical details

The diff updates three core block editor components to conditionally suppress pattern metadata based on the editing context:

- `packages/block-editor/src/components/block-toolbar/block-toolbar-icon.js`: `getBlockIconVariant` now checks `isWithinEditedContentOnlySection( id )` and `isIsolatedEditor` before returning the pattern `symbol`. If either condition is true, it falls back to the actual block type icon.
- `packages/block-editor/src/components/list-view/block-select-button.js`: Removed the `hasPatternName` selector that previously forced the `symbol` icon in the list view. It now directly uses `blockInformation?.icon`.
- `packages/block-editor/src/components/use-block-display-information/index.js`: The hook now evaluates `isWithinEditedContentOnlySection( clientId )` and `isIsolatedEditor` (via `getSettings()?.[ isIsolatedEditorKey ]`). When true, it returns the root block’s title, icon, and description instead of the pattern’s parsed metadata.

Unit tests were added/updated to verify these three states: normal pattern display, edited section display, and isolated editor display.

## Contribution

Opened by @ramonjd in response to issue #78903, building on earlier suggestions by @t-hamano. The PR was reviewed and merged with several co-authors. Follow-up e2e tests were added in PR #79462. The discussion was straightforward, focusing on verifying the UI context switches correctly without breaking existing pattern display behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
