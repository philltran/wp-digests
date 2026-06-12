# #78776: Preserve nested list when deleting a selection across sibling list items

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`5e84a8d`](https://github.com/WordPress/gutenberg/commit/5e84a8dba2c8e16ffd36182d47d5f58aa5fd4a08)
- **Discussion:** [#78776](https://github.com/WordPress/gutenberg/pull/78776) · 3 comments · 0 reactions

## Summary

Fixes a bug in the block editor where deleting a multi-block text selection spanning sibling list items caused nested structures to be dropped. Previously, the merge operation only carried over block attributes from the surviving target block, discarding inner blocks from the deleted block. This update ensures that when two list items are merged via Backspace or Delete, their structural hierarchy is preserved on the resulting merged item.

## Impact

- **Block Editors**: No action required. Resolves unexpected data loss during cross-item text deletion.
- **Plugin/Theme Developers**: If overriding or wrapping `__unstableDeleteSelection`, verify that merge logic now accounts for inheriting `innerBlocks` from the surviving target block to prevent stripping nested structures in custom implementations.
- **Breaking Changes**: None. The affected action is marked `__unstable*` and operates internally within the block editor store.

## Technical details

Modifies `packages/block-editor/src/store/actions.js` inside the `__unstableDeleteSelection` Redux action creator. During the merge phase, the state payload previously constructed only attributes from the target block:
```
{
  ...targetBlock.attributes,
  ...updatedAttributes
}
```
The diff introduces `innerBlocks: blockB.innerBlocks,` to this payload, explicitly retaining inner blocks from the surviving block (`blockB`) while discarding those from the deleted selection range. E2E tests in `list.spec.js` validate preservation across Backspace/Delete keys and multi-line selections.

## Contribution

Opened and merged by @ellatrix as PR #78776. The implementation is a single-line fix focused on carrying over `innerBlocks` from the surviving target block during the merge step of `__unstableDeleteSelection`. Comprehensive E2E specs were added to cover both Backspace and Delete keys across forward/backward selections. No alternative approaches or design debates were raised in the review; the change was accepted as a direct correction to the attribute-only merge behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
