# #81802: Add transform to Row for Columns block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Columns`, `[Feature] Layout`
- **Merged:** [`0b566e4`](https://github.com/WordPress/gutenberg/commit/0b566e44b373c05876fae2b48e2ea86e227b67a3)
- **Discussion:** [#81802](https://github.com/WordPress/gutenberg/pull/81802) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Columns block now has a block transform to the Row variation of Group, and the reverse transform from a flex Row back to Columns. Per-column widths are carried across as flex child sizing (`selfStretch`/`flexSize`) so the layout is reproduced more closely than a naive conversion. It gives authors a one-click migration path from Columns to a more flexible layout block, closing #52857.

## Impact

- **Editors / site owners:** Columns now offers a transform to Row in the block switcher, and a horizontal flex Group can be transformed to Columns. Column widths are preserved as fixed flex sizes.
- **Plugin & theme developers:** No API change and no action required. Code that calls `switchToBlockType( block, 'core/group', 'group-row' )` on `core/columns` will now get a result instead of none. Blocks filtering or overriding `core/columns` transforms should be checked for ordering/priority conflicts.
- **Known limitations (from the PR):** a Columns-level global gap that differs from the Group global gap is not carried over (block-instance gaps are). Styling on the Column block itself, such as background colors, is not copied. Mobile stacking behavior (`isStackedOnMobile`) is dropped, since Row uses its own responsive behavior.
- **Headless / REST / front end:** No effect. This only changes editor-side block transforms.

## Technical details

All changes are in `packages/block-library/src/columns/transforms.js` (plus tests and a CHANGELOG entry).

**Columns → Row** (new `to` entry targeting `core/group` with `variationName: 'group-row'`):
- Builds a `core/group` with `layout: { type: 'flex', flexWrap: 'nowrap', verticalAlignment }`. `verticalAlignment` is kept if it is one of `top|center|bottom|stretch`, else defaults to `'stretch'`. `isStackedOnMobile` and `verticalAlignment` are cleared from the Group's attributes.
- `getRowInnerBlocks()` maps each Column to a Row child. A Column with exactly one inner block is unwrapped, and that block is cloned via `cloneSanitizedBlock` with `style.layout` merged with the child layout. A Column with zero or multiple inner blocks becomes a `core/group` with `layout: { type: 'constrained' }`.
- Widths: numeric widths become `N%`, and string widths containing a digit are used as-is (via `getColumnWidth`). A column with a width gets `{ selfStretch: 'fixed', flexSize: width }`. A column without one gets `{ selfStretch: 'fill' }`. If no column has a width, every column gets an equal fixed width (`+(100 / n).toFixed(2)%`, e.g. `33.33%`).

**Row → Columns** (new `from` entry, `priority: 1`, `isMatch: layout?.type === 'flex' && layout?.orientation !== 'vertical'`):
- Drops `layout` from the attributes and sets `verticalAlignment` only if it is `top|center|bottom` (so `stretch` is dropped).
- `getColumnBlocksFromRow()` wraps each child in a `core/column`. If `selfStretch` is `fixed` or `fixedNoShrink`, `flexSize` becomes the Column `width`. `selfStretch` and `flexSize` are stripped from the child's `style.layout`, and an empty `layout`/`style` is removed. Other layout keys such as `columnSpan` are preserved.

```js
switchToBlockType( columnsBlock, 'core/group', 'group-row' );
switchToBlockType( rowGroupBlock, 'core/columns' );
```

## Contribution

Authored by @tellthemachines as a companion to the existing Grid transform, using Codex-assisted code that the author states they reviewed. The record shows a single design exchange: a reviewer asked whether mobile styles could be injected for a closer 1:1 transform, and the author replied that the transform targets people wanting finer-grained responsive behavior, so stacking parity isn't needed, though it could be added later. Reviewers credited in the bot list include @ramonjd and @andrewserong.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
