# #79103: Image block: don't show crop icon while image is uploading

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Image`, `Backport to Gutenberg RC`
- **Merged:** [`661f47b`](https://github.com/WordPress/gutenberg/commit/661f47bfec5b7b110707da8fdc1b3e26ceb3bce2)
- **Discussion:** [#79103](https://github.com/WordPress/gutenberg/pull/79103) · 3 comments · 0 reactions

## Summary

Resolves a race condition in the Image block where clicking the crop control during an active upload would incorrectly initialize the cropper with previous attachment metadata. The change suppresses the crop UI until the upload lifecycle completes, ensuring editor state aligns with media processing.

## Impact

- **Block & Plugin Developers**: Internal state gating now includes the `isUploading` flag for image controls. No public API changes, deprecations, or breaking modifications.
- **Site Owners/Editors**: Rapid uploads no longer trigger stale cropper instances; clicking crop during upload is silently ignored until processing finishes.
- **Action Required**: None. This is a self-contained block library fix with no migration steps needed.

## Technical details

In `packages/block-library/src/image/image.js`, the conditional rendering check for the image crop/resize tool was extended to guard against an active upload state. The original evaluation:
```javascript
isSingleSelected && canEditImage && ! isEditingImage && ! isContentOnlyMode;
```
Now appends a strict lifecycle guard:
```javascript
isSingleSelected && canEditImage && ! isEditingImage && ! isContentOnlyMode && ! isUploading;
```
This prevents the cropper component from mounting when `isUploading` is true, eliminating race conditions where stale attachment data could leak into the editor UI.

## Contribution

Authored by @ramonjd and co-authored by @andrewserong, this PR addresses a verified UI race condition in the block editor. Following standard review, it was merged as a lightweight patch to the `@wordpress/block-library` package. The fix mirrors existing upload-state gating patterns already implemented in the Site Logo block to maintain consistent editor behavior across media components.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
