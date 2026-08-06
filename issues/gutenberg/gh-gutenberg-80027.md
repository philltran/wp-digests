# #80027: Blocks: Support static block templates in block type settings

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Blocks`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`01c037b`](https://github.com/WordPress/gutenberg/commit/01c037b1ff569b3a91811b26123208e0a1435978)
- **Discussion:** [#80027](https://github.com/WordPress/gutenberg/pull/80027) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Blocks can now declare `template` and `templateInsertUpdatesSelection` directly in the settings object passed to `registerBlockType()` instead of passing them as props to `<InnerBlocks>` or `useInnerBlocksProps`. This shifts template resolution from post-mount synchronization to pre-dispatch insertion, eliminating Real-Time Collaboration (RTC) duplication where multiple connected editors would each independently insert the same template, resulting in N copies of a block.

## Impact

- **Plugin & theme developers:** Migrate `template` and `templateInsertUpdatesSelection` from `<InnerBlocks>`/`useInnerBlocksProps` props to the block type settings passed to `registerBlockType()`. The legacy props are deprecated (since 7.2) but remain functional for backward compatibility.
- **Block authors:** No immediate action required; existing blocks continue to work. Core blocks with static templates have already been migrated.
- **RTC users:** Inserting blocks with static templates will no longer cause duplicate content across connected editors.
- **No action required** for sites not using Real-Time Collaboration or custom blocks with inner block templates.

## Technical details

The diff introduces `applyBlockTypeTemplates()` and `selectBlockTypeTemplate()` in `packages/block-editor/src/store/actions.js`. These functions are invoked inside `insertBlocks` and `replaceBlocks` before dispatching `INSERT_BLOCKS` or `REPLACE_BLOCKS`, ensuring templates are resolved and applied synchronously. `useInnerBlockTemplateSync()` in `packages/block-editor/src/components/inner-blocks/use-inner-block-template-sync.js` now falls back to `getBlockType( getBlockName( clientId ) )?.template` and `?.templateInsertUpdatesSelection` when the legacy props are `undefined`. The legacy props trigger `deprecated()` warnings (since 7.2) but retain backward compatibility. Recursive template application is guarded against infinite loops via an `expandedTypes` Set. Selection is moved to the first inner leaf block only when `templateInsertUpdatesSelection` is `true` and the block is selected.

## Contribution

Opened and merged by @ellatrix with co-authors @jsnajdr, @youknowriad, and @alecgeatches. The change addresses a long-standing RTC duplication issue by shifting template resolution from post-mount synchronization to pre-dispatch insertion. The approach was validated with a dedicated RTC WebSocket test (`collaboration-list-template.spec.ts`) and automated unit tests for the new store actions. No major design debates or rejected alternatives were recorded in the PR thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
