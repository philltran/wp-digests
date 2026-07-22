# #79307: Media: Rename HEIC companion metadata key to source_image

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Feature] Client Side Media`
- **Merged:** [`acb09cb`](https://github.com/WordPress/gutenberg/commit/acb09cb807c04bfc979f0f58e7aef7659ecb9b08)
- **Discussion:** [#79307](https://github.com/WordPress/gutenberg/pull/79307) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg plugin renamed the HEIC companion-file metadata key from `original` to `source_image` to align with WordPress core. This change fixes a silent file-leak bug where the HEIC original would persist on disk after attachment deletion, because core’s cleanup routine reads `source_image` while the plugin was still writing `original`.

## Impact

- **Plugin & theme developers**: The internal metadata key `original` for HEIC companions is effectively deprecated. Update any custom code reading `wp_get_attachment_metadata()['original']` to use `source_image` instead.
- **Site owners & platform teams**: No immediate action required. This prevents orphaned HEIC files from accumulating in the uploads directory once core ships client-side HEIC processing.
- **Breaking change**: Custom attachment-cleanup hooks or metadata parsers that hardcode `original` for HEIC files will break until updated to `source_image`.

## Technical details

The diff updates `lib/media/class-gutenberg-rest-attachments-controller.php` to replace hardcoded strings with class constants `IMAGE_SIZE_SOURCE_ORIGINAL` (`'original-heic'`) and `META_KEY_SOURCE_IMAGE` (`'source_image'`). `finalize_item()` now writes `$metadata[ self::META_KEY_SOURCE_IMAGE ] = $sub_size['file']` instead of `$metadata['original']`, and the `/sideload` route schema, `validate_image_dimensions()`, and `sideload_item()` all reference the constants to prevent schema/metadata drift. In `lib/media/load.php`, `gutenberg_delete_heic_companion_file()` was refactored to read `$metadata['source_image']`, changed its return type to `bool`, and uses `wp_delete_file_from_directory()` for safer cleanup. PHPUnit coverage was added in `phpunit/media/gutenberg-delete-heic-companion-file-test.php` to verify deletion behavior and no-op conditions.

## Contribution

The change was ported directly from the WordPress core backport review to ensure the plugin and core metadata keys stay synchronized before the feature ships. Review focused on alignment and cleanup reliability rather than feature debate, with minimal discussion required to merge the synchronization patch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
