# #82316: Image block cropping: Sync image size and link destination settings

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Block library`, `[Block] Image`, `Backported to WP Core`
- **Merged:** [`9722829`](https://github.com/WordPress/gutenberg/commit/972282952a09e7a999418cb795d9e97073ece990)
- **Discussion:** [#82316](https://github.com/WordPress/gutenberg/pull/82316) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

After cropping, rotating, or flipping an image in the Image block's media editor modal, the block now keeps the user's selected image size (e.g. Medium or Large) instead of rendering the full-size file while the size control still shows the old choice. It also re-points a "Link to image file" or "Link to attachment page" destination at the new attachment instead of the pre-edit image. The fix is in `useOpenImageMediaEditorModal`, and it was a regression introduced with the media editor modal in 7.1.

## Impact

- **Site owners / editors:** Cropping an image set to a non-Full size no longer silently swaps the block to the full-size file. Saved markup and the frontend match the size chosen in the sidebar. Media file and attachment page links now go to the cropped attachment rather than the original.
- **Plugin & theme developers:** No new public API, hook, or block attribute. Image blocks you build or filter around still get `id`, `url`, `sizeSlug`, and `href` updated in a single `setAttributes` call after an edit.
- **Existing content:** Nothing is migrated. Blocks already cropped with a stale size or link are not repaired, and will only be corrected on a re-edit.
- **Backport:** Labeled `Backported to WP Core` and slated for 7.1.1 via Gutenberg #82383. The bug only exists in 7.1.
- No action required beyond picking up the update.

## Technical details

All changes are in `packages/block-library/src/image/use-open-image-media-editor-modal.js`.

**New helpers**
- `getNewAttachmentSizeAttributes( sizeSlug, attachment )` (module-private). If `sizeSlug` is unset or `full`, or the record has no `media_details.sizes`, it returns nothing. Otherwise it returns `{ url: sizes[ sizeSlug ].source_url }`. If the new attachment lacks that sub-size (WordPress skips sub-sizes larger than the file, e.g. after a small crop), it falls back to `{ url: fullUrl, sizeSlug: 'full' }`, where `fullUrl` is `attachment.source_url ?? sizes.full?.source_url`.
- `getNewAttachmentLinkAttributes( linkDestination, attachment )` (module-private). `media` yields `href: attachment.source_url`, which is always the full-size file regardless of the rendered size. `attachment` yields `href: attachment.link`. `custom` and `none` are left alone, and a missing field keeps the existing link rather than clearing it.
- `getNewAttachmentImageBlockAttributes( blockAttributes, attachment )` (exported, used in tests). It merges the two results and returns `undefined` if the attachment record is unknown.

**Hook changes**
- The hook now also reads `sizeSlug` and `linkDestination` from `attributes`, tracks them in `blockAttributesRef`, and adds them to the ref-sync effect dependencies.
- `DEFAULT_MEDIA_SIZE_SLUG`, `LINK_DESTINATION_MEDIA`, and `LINK_DESTINATION_ATTACHMENT` are imported from `./constants`.
- The update callback now computes `isNewAttachment = newId !== currentBlockAttributes.id`. The fresh-record fetch (`resolveFreshAttachmentRecord( newId )`) previously ran only when `originalAttachment` existed. It now also runs when `isNewAttachment` is true.
- If the refetch fails, it falls back to `getCachedAttachmentRecord( newId )`, since the media editor already put the saved record in the store. This avoids half-updating the block, where the new file is shown with the old size and link.
- `onUrlChange` is still raised before anything is awaited. The remaining attributes land in one `setAttributes` call at the end, so the image never renders against half-updated settings. The diff is truncated, so the exact final `onUrlChange` wiring for the size-adjusted URL is taken from the test, which expects `onUrlChange` last called with `cropped-300x200.jpg`.

**Behavior, from the tests**

```js
// attributes: { id: 1, sizeSlug: 'medium', url: 'original-300x200.jpg' }
// media editor reports { id: 2, url: 'cropped.jpg' }
setAttributes( { id: 2, url: 'cropped-300x200.jpg' } );
```

The CHANGELOG entry was added to `packages/block-library`. Jest tests cover the helper matrix, the refetch-failure fallback, the no-baseline case (which invalidates `getEntityRecord` for the new attachment), and media-file link updates. The cropping modal itself continues to use the full source image.

## Contribution

Opened by @andrewserong to fix #82296, with tests and a changelog entry, and Claude Code was disclosed as an AI tool used. The only design discussion was @adamsilverstein asking whether it warranted a 7.1.1 backport, given that the bug was new in 7.1. Andrew judged it non-critical because users can work around it in the UI, but asked for inclusion if possible. The backport was then tracked in #82383.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
