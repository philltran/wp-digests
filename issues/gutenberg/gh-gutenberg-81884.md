# #81884: Media: keep indexed PNG sub-sizes indexed

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Status] In Progress`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`9c92bc7`](https://github.com/WordPress/gutenberg/commit/9c92bc7b0a656a2e287bb6b7fc593efd23db1b88)
- **Discussion:** [#81884](https://github.com/WordPress/gutenberg/pull/81884) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Client-side media processing (`@wordpress/vips`) was re-encoding indexed (palette) PNGs as truecolour when generating sub-sizes, so `large` and intermediate sizes could be several times larger than the original (216KB and 444KB from a 200KB source, versus 70KB and 111KB from server-side Imagick). The fix asks libvips `pngsave` to quantise back to a palette when the source was indexed, on the resize, convert/compress, and rotate paths. Truecolour PNGs are never quantised. On the ticket's image, the 1024px and 1536px sub-sizes drop to 59KB and 114KB.

## Impact

- **Site owners:** Uploading indexed PNGs through the block editor's client-side media pipeline no longer bloats generated sub-sizes. This is a regression in 7.1 that server-side processing had already avoided. No configuration needed.
- **Plugin/theme developers:** No hooks or public signatures change. If you assert on PNG output bytes or colour type from `@wordpress/vips` (`resizeImage`, `compressImage`, `convertImageFormat`, `rotateImage`), indexed sources now yield colour type 3 (palette).
- **`@wordpress/vips` consumers:** PNG output no longer honours the `quality` option (see technical details). This is a no-op in practice because `Q` previously had no effect on PNG.
- **Core/release:** Per the discussion, this is intended for a 7.1 minor release (Trac milestone 7.1.1), and the PR carries the `Backported to WP Core` label.

## Technical details

Changes are in `packages/vips/src/index.ts`:

- New helper `isPaletteImage( image: Vips.Image )` returns `image.getTypeof( 'palette' ) !== 0`. libvips attaches the `palette` metadata field only when the source was indexed, so presence is the signal.
- `buildSaveOptions` now takes an options object `{ type, quality, bitdepth, stripMeta, isPalette }` instead of positional arguments. When `'image/png' === type && isPalette`, it sets `saveOptions.palette = true`. `resizeImage` calls it with `isPalette: isPaletteImage( sourceImage )`.
- `convertImageFormat` sets `saveOptions.palette = true` for PNG output from an indexed source. Converting an indexed source to another format (e.g. WebP) does not set it.
- `rotateImage` reads `isPalette` from the source before the orientation transforms replace `image`, then sets `palette: true` for PNG output. The PR notes CSM cannot reach this path with a PNG today, since the client-side branch is gated to AVIF/HEIF, but the package is published to npm.
- `Q` is no longer passed for PNG. Per the PR discussion, `image/png` was dropped from `supportsQuality()` rather than setting `Q` and deleting it. `Q` is pngsave's quantisation quality and only read once `palette` is on. A lower target produced a larger file on the test fixture (20KB to 30KB).
- `palette` is added to the `SaveOptions` type, and `getSourceBitdepth` is now typed with `Vips.Image` instead of a structural generic.

```ts
// before: palette PNG written as truecolour
image.writeToBuffer( '.png', { strip: true } );
// after: quantised back to the source palette
image.writeToBuffer( '.png', { strip: true, palette: true } );
```

Tests: the vips unit-test mocks now throw for absent metadata fields like real libvips (previously `getInt` returned `0`), so the `try`/`catch` fallbacks are actually exercised. Unit tests cover indexed PNG, truecolour PNG, and non-PNG output. A new e2e test in `client-side-media-processing.spec.js` reads the colour type from the sub-size IHDR chunk, and `rotate-image-integration.ts` has a real-libvips check. The PR also adds a generated fixture, `test/e2e/assets/2000x1200_e2e_test_image_indexed.png`, and a CHANGELOG entry.

Deliberately left alone: pngsave's default dithering. The PR reports `dither: 0` makes no difference for flat artwork and only trades banding for size on gradients, so it's treated as a separate quality decision.

## Contribution

Reported against core as Trac #65922 and tracked in Gutenberg issue #81895, with the PR written with Claude Code assistance by @adamsilverstein. @kleisauke reviewed and steered `isPaletteImage` toward a simple presence check on the `palette` field, and confirmed that requesting `--palette` on write is the intended libvips approach. @swissspidy's feedback led to typing the metadata helpers with `Vips.Image`. Extending the e2e check to every sub-size showed the byte-size assertion can't be looped, because requantising a resized image dithers and the noise costs bytes. @t-hamano asked whether to ship this in the 7.1 minor release, and @adamsilverstein agreed since it is a 7.1 regression and milestoned the Trac ticket 7.1.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
