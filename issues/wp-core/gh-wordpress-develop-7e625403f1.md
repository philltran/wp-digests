# REST API: Sideload external images via a url parameter.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-07-07
- **Commit:** [`7e625403f1`](https://github.com/WordPress/wordpress-develop/commit/7e625403f161e726c78417f924c7345fb2c9bdc4)
- **Usefulness:** 4/5

## Summary

The WordPress REST API now accepts a `url` parameter on the `POST /wp/v2/media` endpoint, allowing the server to fetch and sideload external images directly into the media library instead of requiring client-side browser uploads. This change resolves cross-origin isolation limitations that commonly block direct browser-to-remote-URL requests in modern browsers while preserving the existing multipart upload pipeline for standard file selections.

## Impact

- Plugin & theme developers: Can offload image acquisition to the server via a simple parameter; no changes required for existing `multipart/form-data` uploads.
- Headless & REST consumers: Workflows previously relying on CORS proxies or client-side blob fetches can now use native server-side sideloading, simplifying front-end integrations and reducing browser network constraints.
- Site owners & standard upload flows: No action required; the existing upload control path remains unchanged when `url` is omitted.
- Configuration note: Ensure your hosting environment allows outbound HTTP/HTTPS requests to third-party hosts, as the new endpoint performs server-side downloads.

## Technical details

In `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php`, `get_endpoint_args_for_item_schema()` registers a new `url` argument with `type: 'string'` and `format: 'uri'`. A custom `validate_callback` re-applies `rest_validate_request_arg()` for schema checks, then calls `wp_http_validate_url()` to enforce HTTP/HTTPS schemes and block private or local hosts, mitigating SSRF risks.

When `$request['url']` is present, `create_item()` delegates to a new `protected create_item_from_url( $request )` method. This method extracts the filename from the URL path, validates that the extension maps to an allowed image MIME type via `wp_check_filetype()`, downloads the asset with `download_url()`, and persists it using `media_handle_sideload()`. It explicitly fires `rest_after_insert_attachment` for parity with the standard upload path and returns a 201 response with a `Location` header.

**Pattern shift:**
```php
// Before (client-side blob upload)
const formData = new FormData();
formData.append('file', blob);
fetch('/wp-json/wp/v2/media', { method: 'POST', body: formData });

// After (server-side sideload)
fetch('/wp-json/wp/v2/media', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ url: 'https://external.com/photo.jpg' })
});
```
The existing sub-size generation filters (`image_size_dimensions`, `intermediate_sizes`, etc.) continue to govern derivative creation when `generate_sub_sizes` is left at its default or explicitly set to `true`.

## Contribution

Opened by Adam Silverstein to resolve Trac #65517 and support Gutenberg editor workflows impacted by cross-origin restrictions (referenced in Gutenberg #79407/PR79409). Merged on 2026-07-07 after review feedback from swissspidy, khokansardar, and westonruter. The implementation focused strictly on adding a server-side fetch path while preserving the existing upload controller flow and ensuring all standard REST hooks fire for parity with multipart uploads.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
