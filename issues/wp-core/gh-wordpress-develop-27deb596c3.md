# REST API: Add dimension validation to sideload endpoint.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-07-01
- **Commit:** [`27deb596c3`](https://github.com/WordPress/wordpress-develop/commit/27deb596c302868a509abe8ab9392854d24b49bc)
- **Usefulness:** 4/5

## Summary

This commit enforces strict dimension validation on the `wp/v2/media/<id>/sideload` REST API endpoint, preventing the silent storage of corrupt or dimension-mismatched images. A new internal validator ensures uploaded raster files match the requested image size constraints before metadata is written, rejecting zero/negative dimensions, enforcing exact matches for the `original` size, and capping registered sub-sizes to their configured maximums with a 1px rounding tolerance.

## Impact

- **Headless & REST consumers:** Client-side media processing workflows using the `/wp/v2/media/<id>/sideload` route will now receive explicit 400 `WP_Error` responses (e.g., `rest_upload_dimension_mismatch`, `rest_upload_invalid_dimensions`) instead of silently storing corrupted or oversized files.
- **Plugin & theme developers:** If your code directly calls the attachments controller to sideload raster images, you must handle these new validation failures and ensure payloads do not exceed registered size limits or provide unreadable file streams.
- **No action required** for standard dashboard uploads or themes that do not interact with the REST media sideload flow.

## Technical details

- Introduces `WP_REST_Attachments_Controller::validate_image_dimensions()` and `dimension_exceeds_max()` in `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php`.
- Execution now pivots early in `sideload_item()`: after resolving the file path, it calls `wp_getimagesize( $path )` to extract dimensions before any storage logic runs.
- **Validation rules applied:**
  - Fails immediately if `width <= 0` or `height <= 0`.
  - `'original'` size must exactly match `$metadata['width']` and `$metadata['height']` from the parent attachment.
  - `'full'` and `'scaled'` are exempt from max constraints but still require positive dimensions.
  - Registered sub-sizes (e.g., `thumbnail`, `medium_large`) are checked against registered width/height maximums, allowing a 1px tolerance via `$this->dimension_exceeds_max()` for rounding differences.
- On failure, the uploaded file is cleaned up via `wp_delete_file()` and the endpoint returns a `WP_Error` with status 400, preventing downstream `finalize` operations from processing invalid data.
- Adds PHPStan type hints (`non-negative-int`, typed return for `wp_get_registered_image_subsizes(): array`) to `src/wp-includes/media.php`.

## Contribution

Merged by Adam Silverstein on 2026-07-01 as a follow-up to commit [62428], fixing #64798. The implementation was developed with input from apermo and westonruter. Comprehensive PHPUnit tests were added to `tests/phpunit/tests/rest-api/rest-attachments-controller.php` covering oversize rejections, exact original matches, full/scaled exemptions, and corrupted/unreadable file handling.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
