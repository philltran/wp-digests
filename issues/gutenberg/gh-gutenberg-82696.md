# #82696: Video block: Remove GIF variation handling

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @imraan011
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Video`, `First-time Contributor`
- **Merged:** [`5be0c83`](https://github.com/WordPress/gutenberg/commit/5be0c83f2b00a11abd82770a95d1e775b9c1c0b4)
- **Discussion:** [#82696](https://github.com/WordPress/gutenberg/pull/82696) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Video block's `gif` variation and its companion `isGifVariation` helper are removed entirely. The block no longer registers a `variations` array, the Settings panel (`<InspectorControls>`) is now rendered unconditionally instead of being hidden for GIF-variation blocks, and the `<video>` element's `autoPlay`, `loop`, `muted`, and `playsInline` props are driven by the block's actual attribute values rather than being forced by an `isGif` boolean. This resolves an accessibility issue where GIF-variation videos had no way to enable playback controls.

## Impact

- **Plugin & theme developers:** The named export `isGifVariation` from `packages/block-library/src/video/variations.js` is removed, and the `variations` key is no longer present in the Video block's registered settings. Any code that imported that helper or iterated the video block's variations will break. The `gif` variation (scoped to `['block', 'transform']`) is no longer available in the block switcher.
- **Site editors / content authors:** The Settings panel in the sidebar is now always visible for Video blocks, including those previously identified as GIFs. Animated GIFs inserted via the Media Library or Image→Video transform will still default to `controls: false`, but the user can now toggle controls back on.
- **No action required** for developers who do not reference the removed `isGifVariation` export or the video block's `variations` array.

## Technical details

Three files are modified and two are deleted:

**Deleted:** `packages/block-library/src/video/variations.js` (contained the `isGifVariation` named export and the default-exported `variations` array with `video` and `gif` entries) and `packages/block-library/src/video/test/variations.js`.

**`packages/block-library/src/video/index.js`:** Removed the `import variations from './variations'` line and the `variations` key from the `settings` object passed to `registerBlockType`.

**`packages/block-library/src/video/edit.jsx`:**
- Removed `import { isGifVariation } from './variations'`.
- Destructured `autoplay`, `loop`, `muted`, `playsInline` from `attributes` (previously only `id, controls, poster, src, tracks, width, height` were destructured).
- Removed the `useEffect` that called `videoPlayer.current?.play()` when `isGif` was true.
- Removed the `{ ! isGif && ( … ) }` wrapper around `<InspectorControls>`, making the Settings panel always render.
- Changed the `<video>` element props from hardcoded `isGif` values to the actual attribute values:

```jsx
// Before
autoPlay={ isGif }
loop={ isGif }
muted={ isGif }
playsInline={ isGif }

// After
autoPlay={ autoplay }
loop={ loop }
muted={ muted }
playsInline={ playsInline }
```

The PR description also mentions checking `media.media_details?.animated_video === true` in `onSelectVideo` and in `packages/block-library/src/image/transforms.js`, but those changes are not present in the merged diff.

## Contribution

Opened by first-time contributor @imraan011, closing issue #81943. Reviewers @t-hamano and @adamsilverstein were tagged. @t-hamano left four review comments (redundant `attributes.` prefix, removal of `variations.js` and its test file, and an unnecessary animated GIF check); @imraan011 addressed all four in a follow-up. @adamsilverstein confirmed the behavior matched the ticket's proposal and reported successful manual testing before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
