# REST API: Bound the size of media sideloaded from a URL.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-08-04
- **Commit:** [`5f5d96bd4b`](https://github.com/WordPress/wordpress-develop/commit/5f5d96bd4b0ad25b803dc507c08b5336fff926ed)
- **Usefulness:** 4/5

## Summary

The REST API media creation endpoint now enforces the site's maximum upload size when sideloading files from a remote URL. Previously, single-site installations had no ceiling on this path because `check_upload_size()` skips validation when `! is_multisite()`. This fix caps the remote download at `wp_max_upload_size()` and returns a `400` error if the fetched file exceeds the limit, preventing unbounded disk writes and memory exhaustion.

## Impact

- **Plugin & theme developers / headless consumers:** No breaking changes to the REST schema or existing parameters. The endpoint now returns a `WP_Error` with code `rest_upload_file_too_big` (status `400`) when a remote URL points to a file larger than the configured upload limit.
- **Hosting & platform teams:** No configuration changes required. The limit automatically respects existing `upload_max_filesize`, `post_max_size`, and WordPress `upload_size_limit` filters.
- **Site owners:** No action required. The behavior now aligns with standard media upload limits across all installation types.

## Technical details

The change modifies `WP_REST_Attachments_Controller::create_item_from_url()` in `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php`. It calculates `$max_size = (int) wp_max_upload_size()` and, when greater than zero, attaches a filter to `http_request_args` that sets `limit_response_size` to `$max_size + 1` before calling `download_url()`. After the download completes, the filter is removed. A new size validation block checks `wp_filesize( $tmp_file ) > $max_size`; if exceeded, it deletes the temporary file via `wp_delete_file()` and returns a `WP_Error` with code `rest_upload_file_too_big` and status `400`. The existing multisite `check_upload_size()` call remains unchanged and runs first.

## Contribution

Tracked in ticket #65517 as a follow-up to prior media sideloading hardening patches. The approach was reviewed by andrewserong and courane01, who confirmed that leveraging the existing `http_request_args` filter and `wp_max_upload_size()` was the most efficient way to close the single-site gap without introducing new configuration surfaces or altering the multisite validation path.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
