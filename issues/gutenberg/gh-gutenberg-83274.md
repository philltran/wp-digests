# #83274: Columns: Allow dropping a column at the outer edges

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Feature] Drag and Drop`, `[Package] Block library`, `[Block] Columns`, `[Package] Block editor`
- **Merged:** [`dc51906`](https://github.com/WordPress/gutenberg/commit/dc5190649b9cff181b0e22c1b492b93fc48e0048)
- **Discussion:** [#83274](https://github.com/WordPress/gutenberg/pull/83274) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Columns block now accepts drag-and-drop insertion before the first column and after the last column. Previously, dragging a column over the leading or trailing edge of an outer column produced no insertion indicator, making it impossible to move a column to either end without using List View. The fix passes the Column's root element as a `dropZoneElement` to `useInnerBlocksProps` and extends `useBlockDropZone` to validate drops against the parent block when the target block itself is not a valid drop target.

## Impact

- **Site owners / editors:** Dragging a column to the far left or far right of a Columns block now shows an insertion line and reorders correctly. No configuration or migration needed.
- **Plugin & theme developers:** No API, hook, or `block.json` changes. The `core/column` block's `parent` declaration (`[ "core/columns" ]`) is unchanged. No action required.
- **Hosting & platform:** No action required. Bundle size increases by 73 B total (59 B in `block-editor`, 14 B in `block-library`).

## Technical details

Two files carry the behavioral change.

**`packages/block-library/src/column/edit.jsx`** — The Column edit component now captures its root DOM node in state and forwards it to the inner-blocks drop zone:

```jsx
const [ dropZoneElement, setDropZoneElement ] = useState( null );
const blockProps = useBlockProps( {
  ref: setDropZoneElement,
  // …
} );
// …
const innerBlocksProps = useInnerBlocksProps( { … }, {
  dropZoneElement,
  // …
} );
```

**`packages/block-editor/src/components/use-block-drop-zone/index.js`** — The `useBlockDropZone` hook gains a parent-target fallback. When `dropZoneElement` is set, it computes `isParentDropTargetValid` by calling `isDropTargetValid` against the parent block's type, allowed blocks, and name. The early-return guard changes from:

```js
if ( ! isBlockDroppingAllowed ) { return; }
```
to:
```js
if ( ! isBlockDroppingAllowed && ! isParentDropTargetValid ) { return; }
```

A second guard after position computation rejects any operation other than `'before'` or `'after'` when the target block itself is not a valid drop target, preventing a disallowed block from being inserted into the column. The `dropZoneElement` passed to `getDropTargetPosition` is conditionally set to `undefined` when `isParentDropTargetValid` is false, so the before/after edge-detection logic only activates for the parent-list insertion case.

A minor refactor replaces `getBlockNamesByClientId( [ targetRootClientId ] )[ 0 ]` with the direct `getBlockName( targetRootClientId )` selector.

Three new Playwright e2e tests in `test/e2e/specs/editor/blocks/columns.spec.js` cover: dropping before the first column, dropping after the last column, and confirming that a block disallowed inside a column (e.g. a Paragraph) still drops *into* the column rather than being rejected outright.

## Contribution

Opened by @Mamaduka, who noted the PR was AI-assisted (Claude). @andrewserong reviewed and raised a question about stacked drop zones; Mamaduka acknowledged the pattern is unusual but noted Columns have a single fixed layout direction (unlike the editor, which can swap orientation), and expressed a preference to avoid hardcoding direction-specific logic, hoping the Grid block will eventually replace Columns. The PR was merged at `dc51906` with no further substantive debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
