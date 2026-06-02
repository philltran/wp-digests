# #78713: Try allowing transforms to a variation of another block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Feature] Blocks`, `[Package] Blocks`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`d7e9241`](https://github.com/WordPress/gutenberg/commit/d7e92410926642bca189fb73a141fec72eb06ce1)
- **Discussion:** [#78713](https://github.com/WordPress/gutenberg/pull/78713) · 8 comments · 1 reactions

## Summary

This PR extends block transforms so a block can be converted not just into another block type, but into a *specific variation* of another block. The core mechanism is a new `variationName` argument threaded through `switchToBlockType`. As a demonstration of the capability, it adds a "transform to Grid" option to both the Columns and Gallery blocks, converting them into the Grid variation of `core/group` to change their layout type. It fixes #70355.

## Impact

**Block & theme developers**
- New capability: block `transforms` can now target a named variation of a destination block, rather than only the bare block type. This is primarily useful for transforming into specific `core/group` variations (e.g. Grid) to switch layout type.
- The transform plumbing gains an optional `variationName` parameter on `switchToBlockType`; existing transforms that don't pass it are unaffected.

**Site owners / editors**
- New transform options appear: a multi-column Columns block and a multi-image Gallery block can be transformed into a Grid.

**No action required** for existing content or transforms — the `variationName` parameter is additive and optional. The author explicitly flagged a design reservation that this is mainly useful for Group-variation/layout transforms, and noted that enabling different layouts directly on the original blocks might be a better long-term solution.

## Technical details

The change centers on adding a `variationName` parameter to `switchToBlockType` (in the `@wordpress/blocks` package), then passing that value through the block-editor transform UI and store so that the resulting block is initialized as the named variation of the destination block type rather than the plain block. Changes span three published packages — `@wordpress/blocks`, `@wordpress/block-editor`, and `@wordpress/block-library` — consistent with the reported bundle deltas (`blocks` +102 B, `block-editor` +130 B, `block-library` +299 B; +531 B total).

In the block library, new transform definitions were added to the Columns and Gallery blocks that target the Grid variation of `core/group`. Conceptually the new transform shape is:

```js
// A transform can now resolve to a variation of the destination block
transforms: {
  to: [
    {
      type: 'block',
      blocks: [ 'core/group' ],
      // resulting block is created as the named variation
      // (wired through switchToBlockType's new variationName arg)
      transform: ( attributes, innerBlocks ) => /* ... build group + grid variation */,
    },
  ],
}
```

The diff was not included in the source material, so exact file paths and the precise signature ordering of `switchToBlockType( blocks, name, variationName )` cannot be confirmed here beyond what the description states. There is a prior related attempt to convert Gallery to a grid layout in #60022.

## Contribution

Authored and merged by @tellthemachines (merged in commit `d7e9241`), the PR fixes #70355. The author disclosed it was "mostly gpt 5.5/codexed." The thread is thin — the visible discussion is dominated by the automated size-change report (+531 B). The author raised a design reservation that the feature is mainly useful for transforming into Group variations to change layout, and offered an alternative approach in #78716 (adding a Grid variation directly to the Columns block) for comparison; the Columns/Gallery "transform to grid" options here were added largely for testing the new mechanism.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
