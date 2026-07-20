# #80068: Playlist: use PlainText v2 to avoid HTML entities

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`90c0bed`](https://github.com/WordPress/gutenberg/commit/90c0bedca6bed40ba93b7f64671273cd7e8539bc)
- **Discussion:** [#80068](https://github.com/WordPress/gutenberg/pull/80068) · 4 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block’s `title` and `artist` fields now use the `PlainText` v2 component instead of `RichText` with formatting disabled. This resolves a bug where plain string attributes were still routed through RichText’s HTML entity pipeline, causing unexpected markup in saved content. Server-side rendering also switches from `wp_kses_post` to `esc_html` to align with the plain-text nature of these attributes.

## Impact

- **Block & plugin developers:** No immediate action required for standard usage. If you are building custom blocks that extend or replace `core/playlist-track`, verify that your server-side rendering uses `esc_html()` for plain string attributes instead of `wp_kses_post()`.
- **Site owners & editors:** The `title` and `artist` inputs will now accept and store plain text without RichText’s HTML entity encoding, resulting in cleaner attribute values.
- **No breaking API changes** to public block-editor components or REST endpoints.

## Technical details

In `packages/block-library/src/playlist-track/edit.js`, `RichText` is replaced with `PlainText` (imported from `@wordpress/block-editor`) for both the `title` and `artist` fields. The `allowedFormats={ [] }` and `withoutInteractiveFormatting` props are removed, and `__experimentalVersion={ 2 }` is added to ensure the component treats input as plain text. On the server side (`packages/block-library/src/playlist-track/index.php`), `render_block_core_playlist_track()` now assigns `$alt` directly without `wp_strip_all_tags()` and replaces `wp_kses_post()` with `esc_html()` when outputting `$title` and `$artist`. The corresponding Jest mock in `test/edit.js` is updated to reflect the new `PlainText` props.

## Contribution

Opened and merged by @ellatrix with co-authors @Mamaduka and @mcsf. The PR addresses an alternative approach to #79527, adopting `PlainText` v2 to correctly handle string-type attributes. Review discussion noted internal trade-offs around `PlainText` v1 vs v2 naming and versioning, but the final implementation consistently applies v2 to the Playlist block’s plain-text fields.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
