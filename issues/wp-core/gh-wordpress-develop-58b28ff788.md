# REST API: Enforce multisite upload limits when sideloading media from a URL.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Andrew Serong
- **Committed:** 2026-07-24
- **Commit:** [`58b28ff788`](https://github.com/WordPress/wordpress-develop/commit/58b28ff78834dbec24a29acc2def1753361aadcf)
- **Usefulness:** 4/5

## Summary

The REST API attachments controller now enforces multisite upload limits when creating media via the URL sideload path. Previously, `create_item_from_url()` bypassed `check_upload_size()`, allowing files that exceeded the network-wide maximum file size or a site's upload quota to be uploaded. The fix adds the missing size validation before calling `media_handle_sideload()`, ensuring parity with multipart and raw-body upload endpoints.

## Impact

- **Multisite administrators & hosting teams:** Sites with `fileupload_maxk` or per-site upload quotas will now correctly reject oversized files or quota-exceeded requests via the REST API (`/wp/v2/media` with a `url` parameter).
- **Plugin & theme developers:** Code relying on the REST API to sideload media from URLs will now receive `rest_upload_file_too_big` or `rest_upload_limited_space` errors (HTTP 400) when limits are breached, matching the behavior of the admin media uploader.
- **No action required** for standard single-site installations, as this only affects multisite upload quota enforcement.

## Technical details

The change modifies `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php` in the `create_item_from_url()` method. Before the fix, the method constructed a `$file_array` and passed it directly to `media_handle_sideload()`. The diff inserts a call to `self::check_upload_size( $file_array )` immediately after the temp file is created. If the check returns a `WP_Error`, the temporary file is removed via `wp_delete_file()` and the error is returned to the REST response. This aligns the URL-based creation path with the existing multipart and raw-body upload handlers, which already invoke `check_upload_size()`. The accompanying PHPUnit tests verify that requests exceeding `fileupload_maxk` or the site's space quota now correctly trigger `rest_upload_file_too_big` and `rest_upload_limited_space` error codes.

## Contribution

Andrew Serong authored the patch with review contributions from ramonopoly; the record carries no discussion detail beyond noting it as a follow-up to a prior commit and a direct fix for the reported parity gap.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
