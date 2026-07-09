# #79887: HTML Block: Preserve innerContent when transforming to group, columns

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Feature] Block Transforms`, `[Feature] Drag and Drop`, `[Package] Blocks`, `[Package] Block library`, `[Block] Columns`, `[Block] Quote`, `[Package] Block editor`, `[Block] Group`, `[Block] Widget Group`
- **Merged:** [`519bb65`](https://github.com/WordPress/gutenberg/commit/519bb65e21a53a3c788022c64a6b31991095e62a)
- **Discussion:** [#79887](https://github.com/WordPress/gutenberg/pull/79887) · 11 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This PR fixes a data-loss bug where transforming an HTML block into Group, Columns, Quote, or Widget Group blocks discards the original markup. Previously, block transforms manually reconstructed child blocks using `createBlock()` with explicit attributes and innerBlocks, which ignored the newer `innerContent` parameter used by the HTML block for serialized content. The change centralizes block reconstruction around a sanitization-aware utility that properly preserves all block metadata during transform and drag-and-drop grouping.

## Impact

- **Block & theme/plugin developers:** No action required. The fix operates entirely within core block-editor transform logic.
- **Site owners / editors:** Transforming an HTML block into Group or Columns, or using the Command Center's "Group" action, will no longer strip its content.
- **Widget & legacy migration paths:** Drag-and-drop grouping and wildcard transforms for Widget Group now correctly retain `innerContent`, preventing data loss when Legacy Widgets are converted to core/html blocks.

## Technical details

The unified diff replaces manual `createBlock(name, attributes, innerBlocks)` mapping patterns across five files with a single utility call. Key changes include:

- `packages/blocks/src/api/factory.ts`: Expands the internal `TemplateBlock` type to accept a fourth array element (`innerContent`), and updates `createBlocksFromInnerBlocksTemplate()` to extract and pass this value through to the underlying `createBlock()` call.
- `packages/block-library/src/group/transforms.js`, `columns/transforms.js`, `quote/transforms.js`: Replace `blocks.map(block => createBlock(block.name, block.attributes, block.innerBlocks))` with `__experimentalCloneSanitizedBlock(block)`.
- `packages/block-editor/src/components/use-on-block-drop/index.js`: Applies the same utility to the drag-and-drop row grouping logic that previously rebuilt blocks inline.
- `packages/widgets/src/blocks/widget-group/index.js`: Updates the `__experimentalConvert` wildcard transform to use the new utility, fixing innerContent loss during legacy-to-html conversions.

Before/after pattern (from `group/transforms.js`):
```diff
- const groupInnerBlocks = blocks.map( ( block ) => createBlock( block.name, block.attributes, block.innerBlocks ) );
+ const groupInnerBlocks = blocks.map( ( block ) => __experimentalCloneSanitizedBlock( block ) );
```
Note: The utility is marked `__experimental`, meaning its signature or export stability may change in future releases, but it is now the canonical path for internal transform reconstruction.

## Contribution

Opened by @talldan and merged via commit `519bb65e`. Initial review by @youknowriad identified that the same `innerContent` loss pattern existed in three additional transforms (Quote wildcard, drag-and-drop grouping, Widget Group), prompting an immediate scope expansion. During iteration, the author questioned whether `cloneBlock` adequately sanitizes attributes compared to `createBlock`, but resolved the edge case with internal widget IDs by adopting `__experimentalCloneSanitizedBlock()`. Co-authored by @talldan, @youknowriad, @andrewserong, and @tellthemachines.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
