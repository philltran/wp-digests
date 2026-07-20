# #80071: Generalize playlist block wording

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jeryj
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`59ba9db`](https://github.com/WordPress/gutenberg/commit/59ba9db6203a9a554cc29f08701eb54e3ecde038)
- **Discussion:** [#80071](https://github.com/WordPress/gutenberg/pull/80071) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block's editor UI and frontend rendering strings have been generalized to remove music-specific terminology. Labels such as "Album cover" and "Length" are now "Track image" and "Duration", and internal utility functions were renamed to match. This makes the block's interface more applicable to non-music audio content without altering its underlying behavior or data structure.

## Impact

- **Plugin & theme developers:** No breaking changes to block attributes, `block.json`, REST schema, or PHP render functions. The internal utility `getAlbumCoverAttributes` was renamed to `getTrackImageAttributes` in `packages/block-library/src/playlist/utils.js`, but it is not part of the public block API and does not require migration.
- **Site owners & editors:** UI labels in the block editor sidebar and screen-reader text on the frontend will update to the new wording upon upgrade. No configuration or migration required.
- **Translators:** New strings and updated translator comments require translation updates in GlotPress.

## Technical details

The diff modifies `packages/block-library/src/playlist/edit.js`, `packages/block-library/src/playlist-track/edit.js`, `packages/block-library/src/playlist/utils.js`, and their PHP counterparts (`index.php`). Key changes include:
- Renaming `getAlbumCoverAttributes` to `getTrackImageAttributes` in `utils.js` and updating all call sites, including the `getTrackAttributes` spread.
- Updating `__()` strings in the editor: `Album cover image` → `Track image`, `Length:` → `Duration:`, and normalizing capitalization for `Show tracklist`, `Show artist name in tracklist`, `Show track numbers in tracklist`, and `Show track duration in tracklist`.
- Updating PHP render output in `playlist-track/index.php` to echo `Duration:` instead of `Length:` inside the `.screen-reader-text` span.
- Adjusting test fixtures and assertions in `test/edit.js` and `phpunit/blocks/render-block-playlist-test.php` to reflect the new terminology.
- No changes to block registration, attributes, or the REST API.

## Contribution

Opened and merged by @jeryj with co-authorship from @scruffian. The PR was reviewed and merged with minimal discussion, focusing on aligning the block's terminology with broader audio use cases. CI reported unrelated flaky e2e tests, but the diff itself was straightforward and required no design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
