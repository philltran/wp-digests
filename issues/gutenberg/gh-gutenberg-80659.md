# #80659: Media REST API: Backport sideload from url path upload size check

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Feature] Media`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`8417941`](https://github.com/WordPress/gutenberg/commit/8417941f3e1d3144c3d9526055fd6c0676e8ed3c)
- **Discussion:** [#80659](https://github.com/WordPress/gutenberg/pull/80659) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Media REST API endpoint now enforces multisite upload size limits and space quotas when sideloading files via the `url` parameter. Previously, this path bypassed the `check_upload_size` validation that applies to multipart and raw-body uploads, allowing oversized files to be downloaded and stored even when multisite restrictions are in place. This backport aligns the URL sideload behavior with Core's upload validation logic to prevent quota bypasses.

## Impact

- **Multisite administrators & platform teams:** URL-based media uploads (e.g., the Image block's "Upload to media library" button) will now correctly reject files that exceed site upload limits or available space, returning standard 400 error responses.
- **Plugin & theme developers:** No breaking changes to public APIs or REST schema. Existing integrations using the `/wp/v2/media` endpoint with a `url` parameter will simply receive validation errors instead of silently bypassing size checks.
- **No action required** for standard implementations; the validation now happens automatically within the controller.

## Technical details

In `lib/media/class-gutenberg-rest-attachments-controller.php`, the `Gutenberg_REST_Attachments_Controller::create_item_from_url()` method now validates file size before processing. Previously, the method passed the `$file_array` directly to `media_handle_sideload()`. The diff inserts a validation step:

```php
$size_check = self::check_upload_size( $file_array );
if ( is_wp_error( $size_check ) ) {
    if ( file_exists( $tmp_file ) ) {
        wp_delete_file( $tmp_file );
    }
    return $size_check;
}
```

This ensures multisite `fileupload_maxk` limits and `upload_space_check_disabled` quotas are enforced. The accompanying PHPUnit tests in `phpunit/media/class-gutenberg-rest-attachments-controller-test.php` mock HTTP requests to verify that oversized files trigger `rest_upload_file_too_big` and `rest_upload_limited_space` error codes with a 400 status.

## Contribution

The change originated as a Core parity requirement and was backported to Gutenberg ahead of the WP Core release. @andrewserong authored the implementation, with @ramonjd and @noruzzamans contributing to the review and testing. The PR moved quickly through review, with automated checks flagging unrelated flaky tests that were noted but did not block merge. The commit was subsequently cherry-picked to the `wp/7.1` branch to align with the upcoming release cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
