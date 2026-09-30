# #81606: Inner Blocks: Fix bug that caused alignment controls to show in Image blocks within a Gallery

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Image`, `[Block] Gallery`, `[Package] Block editor`
- **Merged:** [`55dabd6`](https://github.com/WordPress/gutenberg/commit/55dabd69d6141725d2b60482e23d5cd72641884b)
- **Discussion:** [#81606](https://github.com/WordPress/gutenberg/pull/81606) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`InnerBlocks` now resolves the `default` key of a block's `layout` support before handing it to child blocks. Previously, blocks that declare their layout only as a support default (notably Gallery, `supports.layout.default = { type: 'flex' }`) passed the raw support config object to children, which has no `type`. Children then fell back to the flow layout, so an Image inside a Gallery showed left/center/right alignment controls that a flex layout does not permit. The fix is effectively one line in `UncontrolledInnerBlocks`.

## Impact

- **Site owners / editors:** Image blocks nested in a Gallery no longer show the "Align block" toolbar control. Images in normal post content or inside a Group are unchanged.
- **Plugin & theme developers:** Blocks that declare `supports.layout.default` now provide a real layout object (e.g. `{ type: 'flex', ... }`) via layout context to their children, rather than the whole support config. Child blocks that read the layout context and relied on config keys like `allowSwitching` or `allowSizingOnChildren` from the context object would see different values. Blocks in core that declare a `default` layout are Buttons, Columns, Comments Pagination, Gallery, Navigation, Query Pagination, Social Links and Tab List. The PR author checked these and found no consumer expecting the old shape, but flagged it as worth a second look.
- **Custom blocks with nested children:** If your block sets a layout support default and has children that inspect the layout context, verify they behave as expected after this change.
- No migration or configuration change is required. No API was added, removed, or deprecated.

## Technical details

The change is in `packages/block-editor/src/components/inner-blocks/index.js`, inside `UncontrolledInnerBlocks`.

The block's `layout` support is a config object, for example `{ allowSwitching: false, default: { type: 'flex' } }`. The actual default layout lives under `.default`. The old code used the whole config object as the fallback layout:

```js
// before
const usedLayout = layout || defaultLayoutBlockSupport;

// after
const usedLayout =
	layout || defaultLayoutBlockSupport.default || EMPTY_OBJECT;
```

Because the config object has no `type`, the layout context given to children lacked one, and consumers such as the Image block resolved it to the flow layout, which includes alignment controls. With `.default`, children receive the flex layout and the alignment control is hidden. The fallback to `EMPTY_OBJECT` covers supports that have no `default`.

Note that `allowSizingOnChildren` is still read from `defaultLayoutBlockSupport` directly, unchanged by this diff.

The PR also adds `packages/block-library/src/gallery/test/edit.js`, an integration test using `initializeEditor` and `selectBlock`. It asserts that the "Align block" button is absent for an Image inside a Gallery and present for an Image outside one. A `CHANGELOG.md` entry was added under Bug Fixes for `@wordpress/block-editor`. The build-size bot reports a 24 B reduction in `build/scripts/block-editor/index.min.js`.

## Contribution

@youknowriad asked when the regression was introduced. @andrewserong traced it back to #47477, where the design passed the whole `layout` support object to children rather than just the `default` leaf, so it was a long-unnoticed gap rather than a recent regression. He noted that leftover code, such as the check in the Image block's `edit.js` that handles `type` at both the root and under `default`, reflects that structural ambiguity, and asked for a second opinion on whether any downstream blocks depend on the old shape. @tellthemachines asked whether a test for toolbar control visibility was warranted. Andrew justified the lightweight integration test as protection against future layout or Gallery refactors, given his ongoing Gallery work, and the reviewer did not object.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
