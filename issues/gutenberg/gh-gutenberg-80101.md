# #80101: Fix multiple MP3 playlist uploads

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Playlist`
- **Merged:** [`6441f38`](https://github.com/WordPress/gutenberg/commit/6441f384896c255b69ff6cff045dbebd4585bfd1)
- **Discussion:** [#80101](https://github.com/WordPress/gutenberg/pull/80101) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block now correctly handles multiple MP3 uploads and REST-shaped attachment payloads by normalizing media URLs and decoding raw titles. Previously, batch uploads could leave tracks without valid `src` or `title` attributes, and failed track replacements would clear existing source data. This fix ensures uploaded audio files reliably map to block attributes regardless of whether the media provider returns `url` or `source_url`.

## Impact

- **Block & plugin developers**: No code changes required. The Playlist block's internal media handling now accepts both `url` and `source_url` fields and properly decodes HTML entities in attachment titles.
- **Site owners & editors**: Multiple MP3 uploads and track replacements now populate correctly without losing existing track data on failure.
- **No action required** for existing sites or custom blocks that do not directly manipulate Playlist block attributes or rely on the previous media mapping behavior.

## Technical details

The diff modifies `packages/block-library/src/playlist/utils.js` and `packages/block-library/src/playlist-track/edit.js` to normalize incoming media objects. Two new internal helpers were added:
- `getMediaUrl( media )` returns `media?.url ?? media?.source_url`
- `getMediaTitle( media )` returns the title string if present, or decodes `title.raw` / `title.rendered` using `decodeEntities` from `@wordpress/html-entities`

`getTrackAttributes` and `getTrackImageAttributes` now use these helpers instead of directly accessing `media.url` or `media.title`. In `playlist-track/edit.js`, the `onSelectTrack` handler was updated to resolve `mediaUrl` via the same fallback pattern and pass it to `isBlobURL()` and `setTemporaryURL()`.

Before/after usage pattern in `getTrackAttributes`:
```js
// Before
src: media.url,
title: media.title,

// After
src: getMediaUrl( media ),
title: getMediaTitle( media ),
```
This prevents failed replacement uploads from clearing the current track source and ensures REST-shaped attachment payloads are correctly transformed into block attributes.

## Contribution

Opened by @scruffian and merged in commit `6441f38`. The PR was co-authored by @jeryj, @noruzzamans, @getdave, and @fcoveram. During review, @jeryj narrowed the scope to focus specifically on the multiple-upload and attachment-data mapping issues, removing an earlier approach that attempted to enforce upload order via filename matching. The final implementation leverages the existing `onBatchSuccess` signal from `mediaUpload` to batch-process multiple uploads without routing through Gallery-specific logic, while adding targeted fixes for track attribute normalization and replacement failure handling.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
