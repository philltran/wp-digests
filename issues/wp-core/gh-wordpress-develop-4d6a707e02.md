# REST API: Expose size-aware encode quality on attachment responses.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-07-02
- **Commit:** [`4d6a707e02`](https://github.com/WordPress/wordpress-develop/commit/4d6a707e0254329b68d9301a97f177940315b17b)
- **Usefulness:** 4/5

## Summary

This commit introduces an `image_quality` field to the media attachment REST API response, exposing the resolved server-side image encode quality (1–100) based on registered sub-sizes and the `wp_editor_set_quality` filter. The new `wp_get_image_encode_quality()` function in `media.php` replicates the resolution logic of `WP_Image_Editor::set_quality()`, accounting for MIME type defaults (WebP: 86, others: 82), the legacy `jpeg_quality` filter, and dimension-aware overrides. This gives headless consumers accurate quality metadata without needing to instantiate an image editor on the client.

## Impact

- **Headless & REST consumers / Plugin developers**: No immediate action required. The `image_quality` field is read-only and only populated when explicitly requested via standard fields query parameters. Existing clients will safely ignore the new property.
- **Site owners / Platform engineers**: No changes to upload workflows, media library behavior, or image generation. The feature exposes existing filter logic via API without altering server-side encoding.
- **Breaking changes / Deprecations**: None.

## Technical details

- New function `wp_get_image_encode_quality( string $mime_type, array $size = [], ?int $default_quality = null )` added to `src/wp-includes/media.php`. It applies `wp_editor_set_quality`, conditionally applies `jpeg_quality` for JPEG output, clamps out-of-range values to the per-format default, and squashes `0` to `1`.
- Updated `WP_REST_Attachments_Controller::prepare_item_for_response()` in `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php` to compute quality against full-size dimensions and each registered subsize. Only subsizes diverging from the default are returned under a nested `sizes` key.
- Schema updated via `get_item_schema()`: `image_quality` is an object with a static `default` integer (1–100) and a dynamic `sizes` object mapping size names to integers. Context is strictly limited to `'edit'`.
```php
// Response structure added in prepare_item_for_response()
$data['image_quality'] = [
    'default' => 82, // Or 86 for WebP
    'sizes'   => ['thumbnail' => 60] // Only where size-aware filters diverge
];
```

## Contribution

Merged on July 2, 2026, by Adam Silverstein. Props westonruter and timothyblynjacobs for design reviews. The change addresses core ticket #65262 to provide client-side processing with accurate encode quality hints, developed alongside a companion Gutenberg PR (#78420) to support frontend image optimization strategies.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
