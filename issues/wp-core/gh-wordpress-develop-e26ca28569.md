# REST API: Fix sideload and finalize for EXIF rotated images.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-07-20
- **Commit:** [`e26ca28569`](https://github.com/WordPress/wordpress-develop/commit/e26ca285698353181fb103b678774b6b1051d9b7)
- **Usefulness:** 4/5

## Summary

Client-side media uploads via the REST API now correctly handle JPEGs with quarter-turn EXIF orientation (values 5–8). Previously, sideloading a rotated `original` image failed with a `rest_upload_dimension_mismatch` error because the swapped width and height did not match the stored metadata. The fix aligns the `original` sideload path with the `scaled` path: the rotated file becomes the attachment's main file, the unrotated upload is preserved as `original_image`, and the stored EXIF orientation is reset to `1` after rotation.

## Impact

- **Plugin & theme developers / headless clients:** Client-side media processing workflows that sideload rotated images via `/wp/v2/media/{id}/sideload` and `/wp/v2/media/{id}/finalize` will no longer fail with a 400 dimension mismatch error.
- **Platform & hosting teams:** No configuration changes required. The behavior now matches server-side upload rotation, ensuring consistent metadata across upload paths.
- **No action required** for sites not using client-side media processing or EXIF-rotated JPEGs.

## Technical details

The change modifies `WP_REST_Attachments_Controller` in `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php`:
- `validate_image_dimensions()` now accepts transposed dimensions when `image_size` is `original`, acknowledging that EXIF rotation swaps width/height.
- `sideload_item()` merges the `original` and `scaled` branches. It now replaces the attachment's main file with the sideloaded image and stores the previous file in `original_image`, mirroring `_wp_image_meta_replace_original()`.
- `finalize_item()` consolidates `original` and `scaled` handling. It updates `$metadata['width']`, `$metadata['height']`, `$metadata['filesize']`, and `$metadata['file']` from the sub-size payload. Crucially, it resets `$metadata['image_meta']['orientation']` to `1` once the client has applied the rotation, preventing double-rotation on subsequent fetches.

Before/after example for `validate_image_dimensions()`:
```php
// Before
if ( $width !== $expected_width || $height !== $expected_height ) {
    return new WP_Error( 'rest_upload_dimension_mismatch', ... );
}
// After
$matches_dimensions    = $width === $expected_width && $height === $expected_height;
$transposes_dimensions = $width === $expected_height && $height === $expected_width;
if ( ! $matches_dimensions && ! $transposes_dimensions ) {
    return new WP_Error( 'rest_upload_dimension_mismatch', ... );
}
```

## Contribution

The change consolidates previously fragmented handling of `original` and `scaled` image sizes in the REST attachments controller, aligning client-side sideloading with core's server-side rotation logic. Review focused on ensuring metadata parity between client-side and server-side upload paths, with comprehensive PHPUnit tests added to verify transposed dimension acceptance and EXIF orientation reset.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
