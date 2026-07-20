# #79384: Apply and correct EXIF orientation for client side sub-sizes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Client Side Media`
- **Merged:** [`ecb7247`](https://github.com/WordPress/gutenberg/commit/ecb724710e33e559625236201f3a3113ba49d1a3)
- **Discussion:** [#79384](https://github.com/WordPress/gutenberg/pull/79384) · 11 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Client-side media processing now correctly applies and corrects EXIF orientation for AVIF and HEIF sub-sizes when the orientation is stored in an EXIF tag rather than a native `irot`/`imir` transform. Previously, these formats uploaded upright but generated unrotated thumbnails because the server reported `exif_orientation: 1` and libvips/libheif ignore EXIF tags for ISOBMFF containers. The fix adds a client-side ISOBMFF EXIF parser, rotates the source image once via `vipsRotateImage` before sub-size generation, and strips the orientation tag to prevent double-rotation. It also corrects swapped EXIF orientations 5 and 7 in the vips rotation chain.

## Impact

- **Plugin & theme developers / CSM users:** No code changes required. AVIF/HEIF uploads with EXIF-only orientation now generate correctly rotated thumbnails and scaled sub-sizes in the block editor.
- **Hosting & platform teams:** No configuration changes needed. The rotation happens client-side in the browser before upload.
- **Breaking changes:** None. The change is strictly additive to the client-side media pipeline.
- **Note:** An earlier iteration proposed a `media.exifOrientation` JS filter, but it was removed; developers should continue using the server-side `wp_image_maybe_exif_rotate` PHP filter if they need to disable rotation.

## Technical details

- **`packages/upload-media/src/heic-parser.ts`**: Introduces `parseExifOrientation` and `getUnappliedExifOrientation` to read the EXIF Orientation tag (0x0112) from the ISOBMFF `meta`/`Exif` item. `getUnappliedExifOrientation` returns `1` if a native `irot`/`imir` box exists, preventing double-rotation. The `HeicImageData` interface now includes an `exifOrientation` field.
- **`packages/upload-media/src/canvas-utils.ts`**: Adds `applyExifOrientation(source, orientation)` to apply affine transforms for orientations 2–8 directly on an `OffscreenCanvas`. Updated `canvasConvertToJpeg` to fall back to this function when `heicData.rotation` is `0`.
- **`packages/upload-media/src/store/private-actions.ts`**: Rewrites the `generateThumbnails` action to detect `image/avif` or `image/heif` source types. It calls `getUnappliedExifOrientation` on the source file, and if the value is not `1`, rotates the file once via `vipsRotateImage` before generating thumbnails/scaled sub-sizes. The rotated file is sideloaded as the `original_image` metadata.
- **`vipsRotateImage` fix**: Corrects orientations 5 and 7 by changing the operation chain from rotate-then-mirror to `flipHor().rot270()` / `flipHor().rot90()`, matching the EXIF spec. The function now calls `image.remove( 'orientation' )` after rotation to strip the tag.

## Contribution

Opened and merged by @adamsilverstein with co-authorship from @andrewserong. The PR evolved through review cycles where @andrewserong identified swapped EXIF orientations 5/7 and the potential for double-rotation, leading to fixes in `919a6302dcc`. An initial iteration added a `media.exifOrientation` JS filter, but it was removed in `63c6cd6e1d6` after review feedback determined the existing PHP filter adequately covered the disable-rotation use case. The final diff focuses strictly on client-side parsing, rotation, and tag stripping.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
