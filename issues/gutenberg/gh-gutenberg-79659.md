# #79659: Block variations: Support innerContent for the Custom HTML block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @youknowriad
- **Labels:** `[Type] Enhancement`, `[Package] Blocks`, `[Package] Block editor`
- **Merged:** [`7194e37`](https://github.com/WordPress/gutenberg/commit/7194e37af8879685ed7bcf7057d7762510c06c83)
- **Discussion:** [#79659](https://github.com/WordPress/gutenberg/pull/79659) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This pull request adds `innerContent` support to the Block Variations API, enabling developers to preset static HTML markup alongside editable inner-block slots when creating variations of the Custom HTML block (`core/html`). Prior to this change, variations could only define `attributes` and `innerBlocks`, forcing authors to rely on block patterns for complex static/interactive layouts. The enhancement closes a gap in the variations API introduced in #79115.

## Impact

- **Plugin & Theme Developers:** Can now register `core/html` variations that combine fixed HTML structure with editable inner blocks via the standard `wp.blocks.registerBlockVariation()` API.
- **Platform Teams / Other Users:** No action required; this is a non-breaking enhancement to block editor internals and public block APIs. Existing implementations remain fully compatible and require no migration.

## Technical details

The diff modifies `packages/blocks/src/types.ts` to include `innerContent?: Array<string | null>` in the `BlockVariation` interface. It updates `getActiveBlockVariation()` (in `packages/blocks/src/store/selectors.ts`) to accept a new 5th parameter (`innerContent`), implementing deep equality matching via `fast-deep-equal` so variations match when static markup is identical but inner block content differs.

All inserter pathways are updated to extract and forward this field:
- `packages/block-editor/src/store/selectors.js` (`getItemFromVariation`) now carries `variation.innerContent` onto the inserter item object.
- `createBlock()` calls in `use-block-types-state.js`, `block.js` (autocompleter), `inserter-list-item/index.js`, and component previews (`use-styles-for-block.js`, `preview-panel.js`) pass the field as the 4th argument.

UI components that determine active variation state now fetch current block content and supply it to the selector:
- `use-block-display-title.js`, `block-toolbar-icon.js`, and `use-block-display-information/index.js` call `getBlock()?.(clientId)?.innerContent` and pass it through to `getActiveBlockVariation()`.

## Contribution

Merged by @youknowriad with co-authors @priethor and @talldan. The PR addresses a follow-up gap from #79115. During review, @talldan raised concerns about the clunky nature of managing `innerContent`/`innerBlocks` duplication and noted that the resulting block retains its base `core/html` type, making variation identity matching tricky post-insertion. The core team acknowledged these structural limitations but opted to close the immediate API gap first, deferring broader variation-matching architecture improvements to a follow-up spike.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
