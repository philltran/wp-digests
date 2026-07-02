# REST API: Support registering one sideloaded file under multiple image sizes.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-07-01
- **Commit:** [`f7a841388c`](https://github.com/WordPress/wordpress-develop/commit/f7a841388cfa9b445288dfaa4303796584f0a29c)
- **Usefulness:** 4/5

## Summary

The REST API's media sideload and finalize endpoints have been refactored to support efficient client-side media processing. The `image_size` parameter on the sideload endpoint now accepts either a string or an array of strings, allowing clients to register one uploaded file under multiple registered image sizes that share identical dimensions. Instead of synchronously updating full attachment metadata during sideloading, the endpoint returns lightweight sub-size data that clients accumulate and submit in a single batch to the `finalize` endpoint.

## Impact

- **Headless & REST Consumers:** Client-side media processing workflows must adapt to the updated response shape of `/media/<id>/sideload` (returns lightweight metadata instead of full attachment data) and send collected results via a new `sub_sizes` array to `/media/<id>/finalize`.
- **Plugin & Theme Developers:** No direct breaking changes for standard REST API usage. Developers hooking into or extending the sideload/finalize flow should note the shifted metadata update timing, the new batch parameter structure, and the validation callback replacing the schema `enum`.
- **Hosting & Platform:** Reduces redundant image generation, network transfer, and read-modify-write race conditions when clients process identical dimensions across multiple registered sizes.

## Technical details

In `WP_REST_Attachments_Controller::register_routes()`, the `image_size` argument for `/media/<id>/sideload` shifts from a scalar `string` with an `enum` validator to `type: array( 'string', 'array' )`. A custom `validate_callback` replaces the schema-level `enum` to correctly validate both scalars and arrays against dynamically registered sizes.

The `sideload_item()` method now constructs and returns `$sub_size_data` (containing `image_size`, `width`, `height`, `file`, `mime_type`, `filesize`, `original_image`) rather than calling `wp_update_attachment_metadata()` immediately. This architectural shift prevents race conditions during concurrent client-side uploads.

A new `sub_sizes` parameter is added to the `/media/<id>/finalize` route (type: `array`, items: objects matching the `$sub_size_data` schema). The `finalize_item()` method iterates through `$sub_sizes`, applying metadata updates in a single batch. It explicitly supports array-type `image_size` values to register one file under multiple matching size names, and triggers the `wp_generate_attachment_metadata` filter exactly once during finalization.

```php
// Before (synchronous per-sideload)
$metadata = wp_update_attachment_metadata( $attachment_id, $metadata );
return $this->prepare_item_for_response( get_post( $attachment_id ), $response_request );

// After (lightweight return, batch finalize)
$sub_size_data = [ 'image_size' => $image_size ]; // + width, height, file, mime_type, filesize, original_image
return rest_ensure_response( $sub_size_data );
```

## Contribution

Merged on July 1, 2026 by Adam Silverstein (Props: westonruter, swissspidy, sachinrajcp123, sanayasir). The change addresses #65481 and aligns with companion Gutenberg work to enable robust client-side media processing. The diff reflects a deliberate architectural decision to shift from synchronous per-sideload metadata updates to an asynchronous, batch-finalized workflow, validated by new PHPUnit tests in `tests/phpunit/tests/rest-api/rest-attachments-controller.php`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
