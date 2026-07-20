# #80268: Client-side media: generate animated image sub-sizes from the first frame only, matching core

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`4df499f`](https://github.com/WordPress/gutenberg/commit/4df499f95f14f55e86f3eb35a2d205bd22c9b375)
- **Discussion:** [#80268](https://github.com/WordPress/gutenberg/pull/80268) · 13 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The client-side media processing pipeline now generates uncropped sub-sizes of animated GIFs from the first frame only, aligning with WordPress core’s server-side behavior. Previously, the pipeline re-encoded every frame for each uncropped size, causing upload times to exceed 90 seconds and producing sub-sizes larger than the original file. This change also updates the GIF-to-video conversion to use a 10-second keyframe interval, reducing output file size by roughly half without impacting quality or encode time.

## Impact

- **Plugin & theme developers / Client-side media users:** No immediate code changes required. The behavior change is internal to the `@wordpress/vips` and `@wordpress/video-conversion` packages.
- **Site owners / Editors:** Uploads of long animated GIFs will complete significantly faster, and uncropped sub-sizes will be static first-frame stills instead of bloated animated files.
- **Behavioral alignment:** Uncropped animated sub-sizes are now static. This matches core’s `wp_calculate_image_srcset()` guard, which already prevents mixing flattened sub-sizes with animated full-size images, so frontend rendering remains consistent.
- **No action required** for existing implementations; the change is transparent to block usage and only affects the client-side media experiment path.

## Technical details

- In `packages/vips/src/index.ts`, the `resizeImage()` function no longer passes `[n=-1]` to `vips.Image.newFromBuffer()` for uncropped animated images. The previous logic explicitly loaded all frames to preserve animation, but this caused `gifsave` palette re-quantization to dominate processing time. The removal of the `strOptions` variable also simplifies the high-bit-depth AVIF detection path, which previously checked `! strOptions` to determine bit depth.
- In `packages/video-conversion/src/index.ts`, `convertGifToVideo()` now passes `keyFrameInterval: 10` to `mediabunny`'s `VideoSampleSource`, overriding the default 2-second cadence. This sparser keyframe layout halves output size for long animations.
- Before/after snippet for `resizeImage` load options:
  ```javascript
  // Before
  let strOptions = '';
  const loadOptions = {};
  if ( supportsAnimation( type ) && ! resize.crop ) {
    strOptions = '[n=-1]';
    loadOptions.n = -1;
  }
  let image = vips.Image.newFromBuffer( buffer, strOptions, loadOptions );

  // After
  let image = vips.Image.newFromBuffer( buffer );
  ```
- Unit and e2e tests verify that `medium` sub-sizes contain exactly one frame while the full-size attachment remains animated.

## Contribution

Opened and merged by @adamsilverstein, with co-authors @andrewserong and @swissspidy. The PR addresses a performance regression in the client-side media pipeline where long GIFs caused upload queues to stall for minutes. @swissspidy noted the deliberate prior choice to load all frames but agreed alignment with core’s server-side flattening was necessary. Alternatives like tuning `gifsave` parameters or skipping sub-sizes entirely were benchmarked and rejected due to remaining latency or media library fallback issues. The change was backported to the Gutenberg RC and WP Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
