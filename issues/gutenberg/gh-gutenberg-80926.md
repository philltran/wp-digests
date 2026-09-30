# #80926: Playlist: Improve audio conversion and track selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @getdave
- **Labels:** `[Type] Bug`, `Needs Design Feedback`, `[Package] Block library`, `Backported to WP Core`, `[Block] Playlist`
- **Merged:** [`ffd97c0`](https://github.com/WordPress/gutenberg/commit/ffd97c0bbb0c77af2f0b5449b290ed14aae2b19c)
- **Discussion:** [#80926](https://github.com/WordPress/gutenberg/pull/80926) · 16 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block gains block transforms: one or more Audio blocks can be converted into a single Playlist (each becomes a Playlist Track titled with the audio filename), and a Playlist containing exactly one Playlist Track can be converted back into an Audio block. The Playlist's Media Library flows also switch to checkbox-style multi-selection, so users can pick several audio files by clicking them individually without holding Shift or Command.

## Impact

**Site owners / editors**
- Existing Audio blocks can be turned into a Playlist from the Transform menu, and a single-track Playlist can be turned back into Audio.
- Selecting multiple audio files in the Media Library for a Playlist no longer requires modifier keys.
- The Audio-to-Playlist transform only carries over the source, `id`, `blob` and a filename-derived title. Artist, album, duration and artwork are not populated, since Audio blocks don't hold that metadata.

**Plugin & theme developers**
- No breaking changes, deprecations or removed APIs.
- The `core/playlist` block type now exposes `transforms`. Code that inspects or filters possible transformations for `core/audio` or `core/playlist` will see the new entries.
- Playlist Track to Audio is not offered for Playlists with more than one track, or for inner blocks other than a single `core/playlist-track`.
- Note that the Playlist to Audio transform copies only `style.spacing`, so other style keys (e.g. colors, typography) on the Playlist are dropped.

No action required.

## Technical details

**New file `packages/block-library/src/playlist/transforms.js`**, wired into `settings` in `playlist/index.js`.

- **`from` (Audio → Playlist):** `type: 'block'`, `isMultiBlock: true`, `blocks: ['core/audio']`. It creates a `core/playlist` from the first Audio block's attributes (`{ ...attributes[0] }`) with one `core/playlist-track` inner block per Audio block. Each track receives `blob`, `id`, `src` and `title: getFilename( src )` (from `@wordpress/url`). The tests assert that shared attributes such as `align`, `anchor`, `style` and `caption` carry over from the first Audio block, and that a missing `id` is not set on the track.
- **`to` (Playlist → Audio):** `isMatch` requires `block.innerBlocks.length === 1` and that the child is `core/playlist-track`. The transform creates `core/audio` from the Playlist's attributes, keeping only `style.spacing` from `style`, and takes `blob`, `id` and `src` from the track.

**Multi-selection:** in `playlist/edit.js`, the three `MediaPlaceholder` / `MediaReplaceFlow` usages change from `multiple` to `multiple="add"`. This uses the existing media-picker mode where clicking toggles selection additively.

```diff
- multiple
+ multiple="add"
```

The PR adds `test/transforms.js` (Audio→Playlist single and multi-block, layout attribute preservation, missing ID, Playlist→Audio, and no offer for multi-track Playlists) and extends `test/edit-component.js` to assert `multiple === 'add'`. A CHANGELOG entry is added to `packages/block-library`.

## Contribution

The dedicated Playlist Track icon was split out into #80959 during review after @getdave asked design whether the track should have its own icon, and @fcoveram noted @jasmussen already had one in progress. Separately, @fcoveram observed during testing that a track in one Playlist blocks playback of a track in another Playlist, while a standalone Audio block can still play at the same time as a Playlist track; @getdave deferred this to a separate fix and pinged @scruffian and @jeryj. The PR carries the `Backported to WP Core` label.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
