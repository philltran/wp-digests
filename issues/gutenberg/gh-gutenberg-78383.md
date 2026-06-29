# #78383: Commands: add toggle for content-only pattern/template part editing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block editor`, `[Feature] Patterns`
- **Merged:** [`0c605c3`](https://github.com/WordPress/gutenberg/commit/0c605c33a3d45b257e9b12bb5838ee6f8a488497)
- **Discussion:** [#78383](https://github.com/WordPress/gutenberg/pull/78383) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Block Editor now includes a new command palette toggle (`core/toggle-pattern-editing`) that switches unsynced patterns and template parts between content-only mode and fully editable mode within the current session. This addresses usability friction introduced in WordPress 7.0, where these elements defaulted to read-only inner blocks, requiring extra clicks to edit their structure.

## Impact

- **Site editors & pattern authors**: Can toggle full structural editing for unsynced patterns/template parts directly from the command palette (Cmd/Ctrl + K) without clicking through UI overlays.
- **Block editor developers & plugin authors**: The `disableContentOnlyForUnsyncedPatterns` and `disableContentOnlyForTemplateParts` editor settings are now dynamically modifiable via `updateEditorSettings()`. Changing this state immediately halts active content-only sections and alters toolbar/display rendering.
- **No breaking changes or mandatory migrations**: Existing patterns, template parts, and theme.json configurations remain fully functional. No immediate code changes are required unless you programmatically manage pattern editing state.

## Technical details

The diff introduces the `core/toggle-pattern-editing` command in `packages/editor/src/components/commands/index.js`. It reads the current `disableContentOnlyForUnsyncedPatterns` and `disableContentOnlyForTemplateParts` flags from `getEditorSettings()`, then flips both simultaneously via `updateEditorSettings()`. Crucially, it also calls `stopEditingContentOnlySection()` (unlocked from `blockEditorStore`) to immediately terminate any active content-only edit session during the toggle. The new flag state is consumed in `packages/block-editor/src/components/block-toolbar/block-toolbar-icon.js` and `packages/block-editor/src/components/use-block-display-information/index.js`, bypassing the default content-only isolation checks when both flags are enabled. E2E tests verify that inspector headings and toolbar icons correctly reflect the underlying block rather than a pattern placeholder when full editing is active.

## Contribution

Opened by @ramonjd and merged with co-authorship from @talldan. Review focused on simplifying UI copy and ensuring block names display correctly instead of pattern placeholders during mode switches. The author noted planned follow-ups (#79565, #79566) to refine icon display rules for section blocks and move the command into contextual menus when a pattern/template part is selected; those refinements were considered but not included in this merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
