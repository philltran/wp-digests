# #83438: Gallery: Add Order By controls for static gallery, consolidate controls in settings sidebar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Gallery`
- **Merged:** [`ff0659d`](https://github.com/WordPress/gutenberg/commit/ff0659d6eb3e76207aeb63fc97c0c6cdd1fe5e20)
- **Discussion:** [#83438](https://github.com/WordPress/gutenberg/pull/83438) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gallery block's editor sidebar is reorganised: the "Order by" control is moved into the Settings panel for both static and dynamic modes, and static galleries gain a new sorting dropdown (by title or date) that reorders inner image blocks as a one-time action. The previously separate "Randomize order" toggle is folded into the Order By dropdown as a "Random" option. The Source panel is downgraded from a `ToolsPanel` to a plain `PanelBody`, and the "Crop images to fit" toggle is repositioned directly beneath the aspect ratio control.

## Impact

- **Content editors:** Static galleries can now be sorted by title (A→Z, Z→A) or date (newest/oldest) from the Settings panel. The sort is a one-time reorder of inner blocks; dragging images afterwards resets the control to "Custom". No stored attribute records the sort.
- **Plugin & theme developers:** The exported `GallerySourcePanel` component in `packages/block-library/src/gallery/dynamic-gallery.jsx` no longer accepts a `dropdownMenuProps` prop and no longer renders the Order By control internally. Any code importing and rendering this component must drop that prop. This is an internal block-library export, not a documented public API.
- **No action required** for front-end rendering, block attributes, or REST consumers. The `randomOrder` attribute and its front-end behaviour are unchanged.

## Technical details

Three new files are added under `packages/block-library/src/gallery/`:

- **`order-options.ts`** (implied by imports): exports `ORDER_OPTIONS`, `RANDOM_OPTION`, `RANDOM_ORDER`, `parseOrderValue()`, `toOrderValue()`, and the `Order` type.
- **`order-controls.tsx`**: exports two components — `SourceOrderControl` (a `SelectControl` for dynamic galleries that writes `orderby`/`order` into the source query) and `SortImagesControl` (a `SelectControl` for static galleries that triggers a one-time reorder of inner blocks). Both include a "Random" option that calls `onSelectRandom`.
- **`order-images.ts`** (implied by imports): exports `hasSortableImages()` and `sortImageBlocks()`.

In **`edit.jsx`**, the key additions are:

```js
const [ appliedSort, setAppliedSort ] = useState( null );
const currentOrder =
  appliedSort &&
  appliedSort.clientIds.length === innerBlockImages.length &&
  appliedSort.clientIds.every(
    ( id, index ) => innerBlockImages[ index ].clientId === id
  )
    ? appliedSort.order
    : null;
```

`appliedSort` is local `useState` (not a block attribute). It stores the last sort order and the resulting `clientId` sequence. `currentOrder` is `null` (displayed as "Custom") as soon as the inner blocks diverge from that sequence. `sortImages(order)` calls `sortImageBlocks()` then `replaceInnerBlocks()`, and uses `__unstableMarkNextChangeAsNotPersistent()` to fold the `randomOrder: false` write into the same undo level as the reorder.

The old `toggleRandomOrder()` function and its standalone `ToolsPanelItem` are removed. A single new `ToolsPanelItem` labelled "Order by" renders either `SourceOrderControl` or `SortImagesControl` depending on `isDynamic`.

In **`dynamic-gallery.jsx`**, the `OrderControl` component and `ORDER_OPTIONS` constant are deleted. `GallerySourcePanel` switches from `ToolsPanel`/`ToolsPanelItem` to `PanelBody`, and its props shrink from `{ dynamic, dropdownMenuProps, hasImages }` to `{ dynamic, hasImages }`.

## Contribution

Opened by @andrewserong as part of the broader Gallery block sidebar-consolidation effort (tracking issues #15370, #24578, #81370). @ramonjd is credited as co-author. The author noted the change was generated with Claude Code (Fable + Opus 5.5) and then manually reviewed and tested. The author thanked a reviewer with the handle "verbosity" for "eagle eyes." The record carries no further design debate or rejected alternatives beyond the three comments on the PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
