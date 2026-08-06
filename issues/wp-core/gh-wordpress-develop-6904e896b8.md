# REST API: Always register the media creation arguments.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-08-05
- **Commit:** [`6904e896b8`](https://github.com/WordPress/wordpress-develop/commit/6904e896b870e1ddd4e706edcd990b794977fec1)
- **Usefulness:** 4/5

## Summary

The `POST /wp/v2/media` endpoint now unconditionally registers the `url`, `generate_sub_sizes`, and `convert_format` request arguments, regardless of whether client-side media processing is enabled. Previously, these arguments were only registered when the feature was active, which caused unregistered parameters to bypass REST schema validation and sanitization. This fix ensures consistent schema advertising across SSL and non-SSL requests and enforces proper validation (including SSRF protection for the `url` argument) before the request reaches the controller logic.

## Impact

- **Plugin & theme developers / REST API consumers:** No breaking changes to existing payloads. The REST schema for `POST /wp/v2/media` now consistently exposes `url`, `generate_sub_sizes`, and `convert_format` on every request, regardless of the site's SSL configuration or client-side media processing state.
- **Security & platform teams:** The `url` argument now always runs through `sanitize_url` and a custom `validate_callback` that blocks private/local hosts via `wp_http_validate_url()`. Requests that previously bypassed this check and triggered a raw `http_request_failed` error will now correctly return a `400` status with a `rest_invalid_url` error code.
- **No action required** for standard integrations. If you were relying on the previous inconsistent schema behavior or bypassing validation, you will now receive proper validation errors.

## Technical details

The change modifies `WP_REST_Attachments_Controller::get_endpoint_args_for_item_schema()` in `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php`. The conditional `wp_is_client_side_media_processing_enabled()` check wrapping the argument definitions was removed, replacing it with a simple method-type guard:

```php
// Before
if ( WP_REST_Server::CREATABLE === $method && wp_is_client_side_media_processing_enabled() ) {
    // register args...
}

// After
if ( WP_REST_Server::CREATABLE !== $method ) {
    return $args;
}
// register args unconditionally...
```

Additionally, `create_item_permissions_check()` now gates the relaxation of the unsupported image type check behind the same feature flag:

```php
// Before
if ( false === $request['generate_sub_sizes'] ) {
    $prevent_unsupported_uploads = false;
}

// After
if ( wp_is_client_side_media_processing_enabled() && false === $request['generate_sub_sizes'] ) {
    $prevent_unsupported_uploads = false;
}
```

The `url` argument's `validate_callback` explicitly calls `rest_validate_request_arg()` first to enforce `string` type and `uri` format, then applies `wp_http_validate_url()` to reject SSRF vectors, returning a `WP_Error` with status `400` on failure.

## Contribution

Open as a follow-up to #62659 and #62841, this patch addresses the validation gap and schema inconsistency identified in #65808. Adam Silverstein authored the change, with review contributions from andrewserong and jeremyfelt. The team concluded that sideloading, sub-size skipping, and format conversion are all server-capable operations, so gating the schema created unnecessary context-dependent behavior and left the `url` parameter unsanitized. The final implementation unconditionally registers the arguments while preserving the feature flag only where it logically applies to permission checks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
