# #80891: Block editor: make the Group action wrap blocks with a group transform

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`3e9e73c`](https://github.com/WordPress/gutenberg/commit/3e9e73c0fee4c4ec6a470455dfe784c469687ea9)
- **Discussion:** [#80891](https://github.com/WordPress/gutenberg/pull/80891) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Block editor's Group action now correctly wraps selected blocks in a `core/group` block instead of triggering the selected block's own transform to group. Previously, blocks like Quote or Cover that declared a `to` transform for `core/group` would dissolve into their inner blocks when grouped via the toolbar, block options menu, or `Cmd+G` shortcut. This fix ensures the Group action behaves as a structural wrapper, preserving the original block intact inside the group.

## Impact

- **Block & theme developers:** No public API changes or deprecations. Blocks that previously declared a `to` transform for `core/group` will no longer be automatically dissolved when users trigger the Group action. The explicit "Transform to" menu still respects manual transforms.
- **Site owners & editors:** Grouping a Quote, Cover, or similar block now wraps it intact rather than converting it to a paragraph or inner blocks.
- **No migration or configuration required.** The behavioral fix is applied automatically in the editor and does not affect saved post content.

## Technical details

The diff introduces a new `groupBlocks` utility in `packages/block-editor/src/utils/group-blocks.js` and replaces all direct calls to `switchToBlockType` for grouping with this helper across six files: `block-actions/index.js`, `block-tools/index.js`, `convert-to-group-buttons/index.js`, `convert-to-group-buttons/toolbar.js`, `list-view/block.js`, and `use-block-commands/index.js`. 

`groupBlocks` explicitly searches the grouping block's `from` transforms for a wildcard multi-block transform (`transform.type === 'block' && transform.isMultiBlock && transform.blocks.includes('*')`). If found, it invokes `transform.__experimentalConvert` directly, bypassing the selected block's `to` transforms. It falls back to `switchToBlockType` only when no wildcard transform exists. This ensures the Group action's structural intent takes priority over individual block conversion logic. An e2e test in `test/e2e/specs/editor/various/block-grouping.spec.js` verifies that a Quote block with a citation survives grouping intact.

## Contribution

Opened by @ellatrix and merged after review by @ramonjd, @annezazu, @ntsekouras, @carolinan, @colorful-tones, and @bacoords. The PR was backported to the `wp/7.1` branch. A previous attempt to fix this (#70174) tried modifying the Quote block's transform directly, but was abandoned in favor of isolating the Group action's behavior in a dedicated utility. The final approach centralizes the grouping logic without altering existing block transform definitions, keeping the change scoped to the editor's structural actions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
