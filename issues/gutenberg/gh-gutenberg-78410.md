# #78410: Animated GIF to video conversion (via mediabunny)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `Needs Testing`, `[Package] Block library`, `[Feature] Client Side Media`
- **Merged:** [`484eecc`](https://github.com/WordPress/gutenberg/commit/484eecc9bf64569c84bdf5ef2ebd346450fee139)
- **Discussion:** [#78410](https://github.com/WordPress/gutenberg/pull/78410) · 23 comments · 5 reactions
- **Usefulness:** 4/5

## Summary

This PR introduces client-side animated GIF-to-video conversion directly in the browser using WebCodecs (`ImageDecoder` and `VideoEncoder`) and the `mediabunny` library. When an opaque animated GIF is uploaded, it is transcoded to MP4/WebM and stored as a companion file linked to the original attachment. In the editor, the Image block is automatically swapped for a "GIF" variation of the Video block, rendering a native looping `<video>` on the front end without server-side template filtering. This replaces an earlier FFmpeg-WASM pipeline, significantly reducing shipped bundle size while leveraging hardware-accelerated encoding.

## Impact

- **Site owners**: No immediate action required. The editor automatically swaps Image blocks to a looping Video block once client-side conversion completes. Transparent GIFs are excluded from conversion and remain standard images.
- **Plugin & theme developers**: Front-end rendering now outputs a native `<video autoplay loop muted playsinline>` instead of an `<img>` tag when the companion is available. Existing video block filters, templates, and interactivity handlers will apply automatically without modification.
- **Hosting & platform teams**: Media upload workflows will generate two sideloaded companion files (`animated-video` and `animated-video-poster`) per animated GIF attachment. Server-side transcoding is not required; processing occurs entirely in a Web Worker behind Document Isolation Policy.
- **Headless & REST consumers**: The REST API records companion filenames in attachment metadata under `animated_video` and `animated_video_poster`. Clients consuming media structure should account for these new sideloaded resources.

## Technical details

The diff establishes the `@wordpress/video-conversion` worker package, which exposes a Comlink-style proxy that calls `convertGifToVideo(file, mimeType)` off the main thread. Frame extraction uses the browser's native `ImageDecoder` (honoring per-frame delays), and encoding runs via `mediabunny`'s `VideoSampleSource` and `VideoEncoder`. The upload pipeline in `@wordpress/upload-media` triggers this via a new `OperationType.TranscodeGif` operation within `prepareItem()`, gated on WebCodecs availability.

PHP changes are minimal: `lib/media/animated-gif-to-video.php` hooks into `delete_attachment` to safely unlink companion files using `wp_delete_file_from_directory`. The REST controller (`class-gutenberg-rest-attachments-controller.php`) registers `animated-video` and `animated-video-poster` as valid sideload sizes, skipping dimension validation for the video file and recording filenames to `animated_video`/`animated_video_poster` metadata in `finalize_item()`.

In the editor, an Image block containing a converted GIF is swapped to a "GIF" variation of `core/video`, identified by the attribute combination `!controls && loop && autoplay && muted && playsInline`. A dedicated toolbar control (`AnimatedGifConvertControl`) allows authors to revert the block back to the original Image. No render-time PHP filters remain; the video serializes natively.

**Before/After rendering pattern:**
```html
<!-- Before / Fallback (original GIF or pre-conversion state) -->
<img src="animation.gif" alt="" />

<!-- After (editor swap & front-end output) -->
<video autoplay loop muted playsinline poster="animation-poster.jpg" src="animation.mp4"></video>
```

## Contribution

Merged by @adamsilverstein with substantial review from @andrewserong, @swissspidy, and @ramonjd. The implementation architecture shifted significantly during review: initial iterations proposed a render-time PHP filter to swap `<img>` tags to `<video>` elements, but feedback redirected the approach toward an editor-side block switch that moves to a "GIF" variation of `core/video`. This design decision removed ~208 lines of PHP render filters, leaving only attachment cleanup and REST sideload handling on the server. The team also debated packaging structure, settling on a unified `@wordpress/video-conversion` package with `mediabunny` as an internal dependency rather than exposing it as a separate public package.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
