# #78932: List View block support: Hide list tab when allowedBlocks is empty, with no children

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] List View`, `[Package] Block editor`
- **Merged:** [`5286890`](https://github.com/WordPress/gutenberg/commit/5286890e5757196afa99f48e6c2de0a742ba8d91)
- **Discussion:** [#78932](https://github.com/WordPress/gutenberg/pull/78932) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor now hides the List View tab and nested List View panels for any block instance that has no inner blocks and explicitly disallows insertion (`allowedBlocks` is `[]` or `false`). This prevents rendering empty, non-interactive List View UI for blocks using a "managed inner blocks" pattern, such as dynamic galleries that render content via query rather than actual inner blocks.

## Impact

- **Block & plugin developers:** If you use `allowedBlocks: []` or `allowedBlocks: false` on a block with `listView` support and no inner blocks, the List View UI will now automatically hide. No code changes are required; this is a UI behavior adjustment.
- **Theme developers:** No direct impact unless you rely on the List View tab being visible for empty, locked blocks.
- **No action required** for standard block usage or existing themes/plugins.

## Technical details

Introduces `shouldRenderBlockListView( state, clientId )` in `packages/block-editor/src/store/private-selectors.js`. The selector returns `false` when a block lacks `listView` support, *or* when it has zero inner blocks (`getBlockOrder` length is `0`) and `allowedBlocks` is `false` or an empty array. It explicitly preserves the `core/navigation` special case. The new selector replaces inline `hasBlockSupport( name, 'listView' )` checks across four components: `block-card/index.js`, `block-inspector/index.js`, `block-quick-navigation/index.js`, and `hooks/list-view.js`. The logic is evaluated per-instance via `getBlockListSettings`, allowing dynamic toggling without modifying block-type definitions.

## Contribution

Opened and merged by @andrewserong with review contributions from @tyxla, @ramonjd, and @talldan. The PR initially proposed a declarative `listView` flag on `useInnerBlocksProps`, but @tyxla suggested inferring the "managed inner blocks" state from existing settings. The author pivoted to the inferred approach, which simplified the implementation and resolved reviewer concerns about imperative toggles. The `core/navigation` exemption was explicitly preserved after testing revealed edge cases with parent `templateLock: 'contentOnly'` overrides.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
