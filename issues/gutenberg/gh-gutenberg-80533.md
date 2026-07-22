# #80533: Playlist: Fix playback of tracks served without CORS headers

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Block library`, `Backported to WP Core`, `[Block] Playlist`
- **Merged:** [`d23c292`](https://github.com/WordPress/gutenberg/commit/d23c2927c7f0e25dbe656789b5ea79239c96dc43)
- **Discussion:** [#80533](https://github.com/WordPress/gutenberg/pull/80533) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block can now play audio tracks hosted on CDNs or external hosts that do not send CORS headers. Previously, the underlying `@arraypress/waveform-player` library unconditionally added `crossorigin="anonymous"` to the generated `<audio>` element, causing playback to fail with a CORS error. Bumping the dependency to v1.23.0 removes this hardcoded attribute, restoring playback and aligning the Playlist block with the existing Audio block behavior.

## Impact

- **Plugin & theme developers / Site owners:** No action required. The Playlist block will now successfully load and play media from hosts without `Access-Control-Allow-Origin` headers.
- **Headless & REST consumers:** No impact. This is a client-side block rendering fix.
- **Note:** Waveform peak analysis still falls back to placeholder peaks for no-CORS hosts (tracked separately in #80534).

## Technical details

The change bumps `@arraypress/waveform-player` from `^1.21.0` to `^1.23.0` in `packages/block-library/package.json`. In v1.23.0, the library's `createAudio()` method no longer sets `this.audio.crossOrigin = 'anonymous'` by default, making it an optional configuration parameter instead. The PR drops an earlier workaround that manually cleared `audio.crossOrigin` after player instantiation. A new unit test in `packages/block-library/src/utils/test/waveform-utils.js` asserts that `initWaveformPlayer()` produces an audio element with `crossOrigin` set to `null` and no `crossorigin` attribute, guarding against upstream regressions or dependency downgrades.

## Contribution

Opened by @adamsilverstein and merged with co-authors @swissspidy and @scruffian. The PR initially implemented a client-side workaround to strip the `crossOrigin` attribute, but after upstream maintainer @arraypress resolved the issue in `@arraypress/waveform-player@1.23.0`, the workaround was dropped in favor of a direct dependency bump. The change was backported to the `wp/7.1` branch for inclusion in the next WordPress release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
