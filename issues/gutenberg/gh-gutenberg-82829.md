# #82829: Image: Restore alt text and caption when undoing a media editor save

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Block library`, `[Block] Image`
- **Merged:** [`9ea6b81`](https://github.com/WordPress/gutenberg/commit/9ea6b81eb43689edd78a7e2c019bc2f7546a2a88)
- **Discussion:** [#82829](https://github.com/WordPress/gutenberg/pull/82829) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Undoing a media editor save on the Image block (via the "Image edited." snackbar) now restores the block's alt text and caption, not just the image. Previously, a save that created a new attachment would sync that attachment's alt text and caption onto the block, but the Undo only passed back the previous attachment's id and URL, leaving the synced metadata in place. The fix tracks the overwritten values in a ref and restores them when the next update returns the block to the original attachment.

## Impact

- **Site editors / content authors:** No action required. The undo behavior in the Image block's media editor is now correct; alt text and caption revert along with the image.
- **Plugin & theme developers:** No action required. No public API, hook, or block attribute schema changed. The fix is internal to the `useOpenImageMediaEditorModal` hook.
- **No breaking changes, deprecations, or configuration changes.**

## Technical details

The change is entirely within `packages/block-library/src/image/use-open-image-media-editor-modal.js`.

A new ref, `mediaEditorUndoMetadataRef`, stores the attachment id the block moved away from and the block's own `alt`/`caption` values that a save overwrote. The logic:

1. **On save to a new attachment** (`isNewAttachment` is true inside `handleMediaUpdate`): the ref is populated with `{ id: currentBlockAttributes.id, attributes: { alt, caption } }` (the keys come from `resolvedMetadataAttributes`).
2. **On the next update** (the Undo): if `undoMetadata?.id === newId`, the stored attributes are merged into `nextAttributes` via `Object.assign`, so `setAttributes` receives the original alt and caption alongside the restored id and URL.
3. **On modal open** (the `useEffect` that snapshots the baseline): `mediaEditorUndoMetadataRef.current` is set to `undefined`, so a save in a *new* session that happens to return the block to the same attachment is not mistaken for an Undo.
4. **At the top of `handleMediaUpdate`**: the ref is read into a local `undoMetadata` and then cleared, so only the update immediately following the save can trigger the restore.

Before (effective behavior on Undo):
```js
// setAttributes called with only id + url; alt/caption stay at the
// new attachment's synced values
setAttributes( { id: 1, url: 'original.jpg' } );
```

After:
```js
// setAttributes called with id + url + restored alt/caption
setAttributes( { id: 1, url: 'original.jpg', alt: 'Original alt', caption: 'Original caption' } );
```

Two new jsdom tests were added in `packages/block-library/src/image/test/use-open-image-media-editor-modal.jsdom.test.js`:
- `restores the alt text and caption a save overwrote when undoing back to the previous attachment` — verifies the happy-path restore.
- `does not restore overwritten metadata when a later session saves back to the previous attachment` — verifies that opening the modal a second time clears the stored metadata, so a subsequent save to the same attachment uses the server's current values rather than the stale block values.

Bundle impact: +68 B in `build/scripts/block-library/index.min.js`.

## Contribution

Opened by @ramonjd. During review, @andrewserong suggested adding test coverage for the restoring behavior to the existing test suite; @ramonjd agreed and added a test that exercises sync after a save completes (which would fail without the fix). The PR was merged as `9ea6b81` with both contributors credited.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
