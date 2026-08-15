# REST API: Bind finalize sub-size file names to their sideload.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-08-12
- **Commit:** [`3767a6c06a`](https://github.com/WordPress/wordpress-develop/commit/3767a6c06a1695e45c2f4730cc353f42aeb4b730)
- **Usefulness:** 3/5

## Summary

The REST attachments controller's `finalize` endpoint previously stored whatever sub-size file names a client sent, without checking that a sideload had actually produced them for that attachment. Because those names are later resolved against the attachment's upload directory and then read or deleted, a finalize request could record — and later cause core to operate on — a file the upload never created. This changeset makes `sideload_item()` record every file name it generates under a new `_wp_sideloaded_file` post meta key, and makes `finalize_item()` accept a name only if a prior sideload recorded it, it is the attachment's own attached file, or it is already present in the attachment's metadata; anything else fails with `rest_invalid_sub_size_file` (HTTP 400) before any metadata is written. Sideloads are also now pinned to the attachment's own uploads subdirectory via an `upload_dir` filter, so generated names live in the directory they are later resolved against, and the `image_size` / `sub_sizes` schemas are tightened considerably.

## Impact

**Clients of the sideload/finalize media endpoints (headless apps, JS media uploaders, client-side image processing)**

- A `finalize` request may now fail with `rest_invalid_sub_size_file` (HTTP 400) if it submits a `file` name that no sideload on that attachment produced. Flows that synthesized or reused sub-size names client-side must submit only names returned by a prior sideload response for the same attachment.
- Finalize remains idempotent: a name already recorded in the attachment's metadata is accepted on a repeat call.
- `sub_sizes` is now capped at `maxItems: 100`, and every entry's `image_size` is validated with the same enum logic as the sideload endpoint. Empty strings are rejected (`minLength: 1` on `image_size`, `file`, `original_image`; `minItems: 1` on `image_size` arrays).
- Passing an array of size names now only accepts *regular* registered sub-sizes. The special names `original`, `scaled`, the source-format original constant, `animated_video`, and `animated_video_poster` are scalar-only and are rejected inside an array.
- Dimension validation now runs for **every** name in an `image_size` array. Wrapping a name in a one-element array previously skipped `validate_image_dimensions()` entirely; such requests will now be rejected if the file's dimensions don't match.
- New failure mode: sideloading against an attachment whose attached file is not under the uploads directory returns `rest_sideload_attachment_not_in_uploads` (HTTP 403). The former `rest_sideload_no_attached_file` (404) branch is gone — the attached file is now resolved up front.

**Site owners / hosting**

- Sideloaded files are no longer placed in a directory derived from the parent post's date; they go into the same `year/month` subdirectory as the attachment they extend.
- A new per-attachment meta key `_wp_sideloaded_file` accumulates one row per generated file name (via `add_post_meta()`, not a serialized array).

**Everyone else** — no action required; these routes are the in-development client-side media pipeline, marked `@since 7.1.0`.

## Technical details

All changes are in `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php`.

**New meta key.** `const META_KEY_SIDELOAD_FILE_NAME = '_wp_sideloaded_file'` is added alongside the existing `META_KEY_SOURCE_IMAGE`. The docblock states each successful sideload *appends* the name(s) it created via `add_post_meta()` — one row per value — specifically so concurrent sideloads don't read-modify-write a shared value. Finalize reads these back as the allowlist.

**Upload directory pinning.** `sideload_item()` resolves `get_attached_file( $attachment_id, true )` once at the top and derives the attachment's subdirectory via `get_attachment_upload_subdir()`; if either is unavailable it bails with `rest_sideload_attachment_not_in_uploads` (403). The old parent-post-date logic is replaced by an `upload_dir` filter at priority 100:

```php
// Before: derive $time from the parent post, as media_handle_upload() does.
$parent_post = get_post_parent( $attachment_id );
$time        = null;
if ( $parent_post && 'page' !== $parent_post->post_type && ! str_starts_with( $parent_post->post_date, '0000-00-00' ) ) {
    $time = $parent_post->post_date;
}
$file = $this->upload_from_file( $files, $headers, $time );

// After: pin the upload to the attachment's own subdir.
$filter_upload_dir = static function ( $uploads ) use ( $subdir ) {
    if ( is_array( $uploads ) && isset( $uploads['basedir'], $uploads['baseurl'] ) && /* ... */ ) {
        $uploads['subdir'] = $subdir;
        $uploads['path']   = $uploads['basedir'] . $subdir;
        $uploads['url']    = $uploads['baseurl'] . $subdir;
    }
    return $uploads;
};
add_filter( 'upload_dir', $filter_upload_dir, 100 );
$file = $this->upload_from_file( $files, $headers );
// ... remove_filter( 'upload_dir', $filter_upload_dir, 100 );
```

**Size-name validation extracted.** The inline `validate_callback` closure for `image_size` is replaced by two private statics: `validate_image_size_names( $value, $param )` and `get_special_image_sizes()`. The latter returns `original`, `scaled`, `self::IMAGE_SIZE_SOURCE_ORIGINAL`, `animated_video`, `animated_video_poster`. The former builds the regular set from `wp_get_registered_image_subsizes()` plus `full` (not a registered sub-size, but stored as an ordinary `sizes` entry for PDF thumbnails), minus the special names — and only the scalar branch merges the special names back in. Both the sideload `image_size` arg and the new `sub_sizes` callback call `rest_validate_request_arg()` first, because supplying a custom `validate_callback` otherwise replaces the declared schema validation (type, `minLength`, `minItems`).

**sub_sizes validation.** `sub_sizes` gains `maxItems: 100` and a `validate_callback` that iterates entries and runs `validate_image_size_names()` on each `$sub_size['image_size']`, reporting the failing index as `sub_sizes[<i>][image_size]`. The comment notes the callback sits on `sub_sizes` rather than the nested property because a nested schema property cannot carry one.

**Dimension checks.** The array/scalar split in `sideload_item()` is removed: `foreach ( (array) $image_size as $size_name )` now runs `validate_image_dimensions()` for each name, deleting the uploaded file with `wp_delete_file()` on failure. Redundant repeat calls to `wp_getimagesize()` and `get_attached_file()` are dropped in favor of the values resolved earlier, and two previously-leaking failure paths (`rest_sideload_invalid_image`, `rest_sideload_update_attached_file_failed`) now `wp_delete_file( $path )` before returning.

**Typing.** A `@phpstan-type Image_Sub_Size` array shape is added to the class docblock, and `create_item_from_url()` is given a `WP_REST_Request $request` parameter type.

## Contribution

This is the third pass over the same sideload/finalize code path, landing as a follow-up to r61982 and r62609 and tracked under Trac #64804. Weston Ruter reviewed and is credited with props; the record carries no further discussion or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
