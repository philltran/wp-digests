# #78852: Media: Move client-side media compat file to wordpress-7.1 directory

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Code Quality`, `[Package] Core data`, `No Core Sync Required`, `[Feature] Client Side Media`
- **Merged:** [`8105732`](https://github.com/WordPress/gutenberg/commit/8105732436f38356011810ddbe8856452a35344e)
- **Discussion:** [#78852](https://github.com/WordPress/gutenberg/pull/78852) · 7 comments · 0 reactions

## Summary

Internal refactoring moves client-side media compatibility files from the `wordpress-7.0` directory to `wordpress-7.1`, aligning with WordPress's version-compat convention since the feature graduated from experimental but was not shipped in 6.9/7.0. This cleanup ensures obsolete code is properly removed when Gutenberg's supported core version increments, preventing long-term bloat.

## Impact

- **Plugin & theme developers**: No action required. The change affects internal Gutenberg package files and does not expose or alter any public APIs or filters.
- **Site owners / Platform teams**: No action required.
- Note: This is purely an internal code-quality adjustment to the `@wordpress/core-data` and client-side media packages, with zero behavioral exposure to consumers.

## Technical details

- Moved `lib/compat/wordpress-7.0/media.php` to `lib/compat/wordpress-7.1/media.php`.
- Updated the corresponding `require` statement in `lib/load.php` to reference the `wordpress-7.1` path.
- Extracted `gutenberg_block_editor_preload_paths_root_fields()` from `lib/compat/wordpress-7.0/preload.php` into a new `lib/compat/wordpress-7.1/preload.php`. This filter remains in the 7.1 compat directory as a stopgap for pre-7.1 installations, ensuring client-side media preload fields are loaded before core directly authors them in `wp-admin/edit-form-blocks.php` and `wp-admin/site-editor.php`.
- The structural shift aligns the `lib/compat` layout with WordPress's policy: compat files belong in folders matching the actual release version where the feature ships.

## Contribution

PR #78852 was authored by @adamsilverstein and merged as commit `8105732`, incorporating review feedback from @t-hamano. The discussion clarified that `lib/compat` files should reside in directories matching the target WordPress release to prevent accumulating dead code as supported versions bump. While core backported the underlying PHP logic directly to 7.1 (r61703), the Gutenberg-side filter was retained in the 7.1 compat directory as a necessary fallback for older installs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
