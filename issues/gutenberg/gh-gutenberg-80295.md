# #80295: Media REST API: Fix sideload and finalize for EXIF rotated images

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`2059b9c`](https://github.com/WordPress/gutenberg/commit/2059b9c9e24b58a4b06674ce05b91506faf79472)
- **Discussion:** [#80295](https://github.com/WordPress/gutenberg/pull/80295) · 14 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Fixes a bug in the client-side media upload flow where EXIF-rotated images triggered a `400` dimension mismatch error during the `sideload` and `finalize` REST API steps. The change aligns the `'original'` image size handling with the existing `'scaled'` logic, allowing transposed dimensions and resetting the stored EXIF orientation to `1` once the client applies the rotation. This ensures images with orientation tags (e.g., 6, 8) upload successfully without server-side processing.

## Impact

- **Headless & REST consumers:** No breaking changes. The REST API now accepts transposed dimensions for `'original'` sub-sizes during client-side uploads, resolving a previous `rest_upload_dimension_mismatch` error.
- **Plugin & theme developers:** No migration or code changes required. The controller now handles client-applied rotations consistently with scaled images.
- **Site owners / Editors:** Images that previously failed to upload or displayed incorrectly in the block editor will now process correctly.
- **Hosting & platform:** No configuration changes required; the fix is contained within the media REST controller.

## Technical details

The unified diff modifies `lib/media/class-gutenberg-rest-attachments-controller.php` in the REST attachments controller:

- **`finalize_item()`**: Merges the `'original'` and `'scaled'` branches. It now validates that `$sub_size['file']` exists, updates `$metadata` fields (`width`, `height`, `filesize`, `file`, `original_image`), and explicitly resets `$metadata['image_meta']['orientation']` to `1` to prevent double-rotation on re-fetch.
- **`validate_image_dimensions()`**: Updates the `'original'` size check to accept transposed dimensions alongside exact matches:
  ```php
  // Before
  if ( $width !== $expected_width || $height !== $expected_height ) { ... }
  // After
  $matches_dimensions    = $width === $expected_width && $height === $expected_height;
  $transposes_dimensions = $width === $expected_height && $height === $expected_width;
  if ( ! $matches_dimensions && ! $transposes_dimensions ) { ... }
  ```
- **`sideload_item()`**: Unifies `'scaled'` and `'original'` handling to record the pre-rotation file as `original_image` and update the attached file pointer, mirroring core's `_wp_image_meta_replace_original()` behavior.
- **Tests**: Added `test_sideload_original_accepts_transposed_dimensions()` and `test_original_sideload_matches_server_side_rotation()` to verify the new validation path and metadata parity with server-side uploads.

## Contribution

Opened by @andrewserong as part of the client-side media upload initiative (tracking #77582). The PR was reviewed and tested by @adamsilverstein and @ramonjd, who verified the fix across JPEG, HEIC, and AVIF formats. The approach of reusing `'scaled'` logic for `'original'` sideloading was discussed in review but accepted as the cleanest way to handle client-applied EXIF rotations without duplicating validation paths. Merged into Gutenberg and backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
