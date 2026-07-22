# #79500: Classic Block: Remove conversion from `BlockInvalidWarning`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tyxla
- **Labels:** `[Type] Enhancement`, `[Block] Classic`, `[Package] Block editor`
- **Merged:** [`3e58de9`](https://github.com/WordPress/gutenberg/commit/3e58de9dba853d8fefb564b606c58d88e99e225a)
- **Discussion:** [#79500](https://github.com/WordPress/gutenberg/pull/79500) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor no longer offers a "Convert to Classic Block" action in the `BlockInvalidWarning` component when a block contains invalid or unexpected markup. This change removes the conversion pathway to `core/freeform` to align with WordPress's ongoing effort to de-emphasize the classic block and steer users toward modern recovery options like "Resolve" or "Convert to HTML".

## Impact

- **Block editor users & theme/plugin developers:** The default invalid block warning UI will no longer display the "Convert to Classic Block" button. This is a behavioral change to the editor's default interface, not a breaking public API change.
- **Developers customizing the editor:** If your code explicitly checks `canInsertClassicBlock` within the `BlockInvalidWarning` context or relies on the legacy conversion flow, you will need to adjust your logic. The underlying `canInsertBlockType( 'core/freeform', … )` selector remains available in `blockEditorStore` for other use cases.
- **No action required** for standard implementations or plugins that do not override the default warning modal.

## Technical details

The diff modifies `packages/block-editor/src/components/block-list/block-invalid-warning.js` to strip out the classic block conversion logic. Specifically:
- Removes `canInsertClassicBlock` from the `useSelect` hook, eliminating the `canInsertBlockType( 'core/freeform', rootClientId )` check.
- Deletes the `convert.toClassic()` method that previously instantiated a `core/freeform` block via `replaceBlock`.
- Filters out the corresponding UI entry from the `secondaryActions` memo.
- Updates the `useMemo` dependency array to remove `canInsertClassicBlock`.

Before:
```js
const { block, canInsertHTMLBlock, canInsertClassicBlock } = useSelect( ... );
// ...
secondaryActions: [
  // ...
  canInsertClassicBlock && {
    title: __( 'Convert to Classic Block' ),
    onClick: convert.toClassic,
  },
].filter( Boolean ),
```

After:
```js
const { block, canInsertHTMLBlock } = useSelect( ... );
// ...
secondaryActions: [
  // ...
].filter( Boolean ),
```

## Contribution

Opened and merged by @tyxla, with co-authorship noted for @Mamaduka. The change was implemented as part of the broader #78067 initiative to de-emphasize the classic block across the editor. Review was straightforward, with no alternative recovery pathways debated; the team aligned on removing the legacy button as a direct cleanup to match the modern inserter's behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
