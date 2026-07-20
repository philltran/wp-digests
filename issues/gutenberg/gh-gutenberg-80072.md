# #80072: GIF to video conversion: make it opt-in. Switching via block transforms

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Image`, `[Block] Video`, `[Feature] Client Side Media`
- **Merged:** [`f0bbefe`](https://github.com/WordPress/gutenberg/commit/f0bbefe6288bf8805c22686414db75e8b556ec4e)
- **Discussion:** [#80072](https://github.com/WordPress/gutenberg/pull/80072) · 13 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Animated GIF uploads in the Image block no longer automatically convert to a Video block once the sideloaded video companion is ready. Instead, the conversion is now opt-in via the block switcher (Transforms API), offering a synchronous switch to the `core/video` block's "GIF" variation. This change eliminates the race-condition-like behavior where the block swapped underneath the author after a background transcode, and removes bespoke toolbar buttons in favor of the standard transforms dropdown.

## Impact

- **Plugin & theme developers:** No public API changes, but internal components `AnimatedGifConvertControl` and the `gif-conversion-attributes.js` utility were removed. Custom code relying on these internals will need to be refactored or removed.
- **Content editors / site owners:** Uploading an animated GIF will now remain an Image block. To play it as a video, editors must manually switch blocks via the block switcher. The reverse switch (Video to Image) is no longer available; editors must use the undo step to revert.
- **No action required** for standard usage. The underlying media processing pipeline (transcoding and sideloading) remains unchanged.

## Technical details

- Removed automatic block replacement in `packages/block-library/src/image/edit.js` that previously triggered when `media.media_details?.animated_video` was present.
- Deleted `packages/block-library/src/image/animated-gif-convert-control.js` and its unit tests.
- Added a `to` transform in `packages/block-library/src/image/transforms.js` that matches when the attachment record contains a sideloaded `animated_video` companion. It creates a `core/video` block with attributes `{ controls: false, loop: true, autoplay: true, muted: true, playsInline: true }` and carries over `align`, `anchor`, `className`, and `style.spacing.margin` via the new `getCarriedGifConversionAttributes` helper.
- The helper `getAnimatedGifVideoCompanion` reads the attachment record synchronously from `@wordpress/core-data` (`select( coreStore ).getEntityRecord`) to support synchronous transform matching.
- The reverse transform (`core/video` to `core/image`) was explicitly removed during review to simplify the feature, relying on the editor's undo history instead.
- E2E tests in `test/e2e/specs/editor/various/gif-to-video.spec.js` were updated to verify the opt-in flow and undo behavior.

## Contribution

Opened by `@adamsilverstein` as a follow-up to #78410, addressing design feedback from `@jasmussen` and `@Mamaduka` regarding the automatic swap's race conditions and toolbar space usage. During review, `@Mamaduka` suggested removing the reverse transform and unit tests, which `@adamsilverstein` implemented. `@andrewserong` noted the UX value of a reverse transform but acknowledged the simplification. The PR was merged with `@andrewserong`, `@Mamaduka`, and `@jasmussen` as co-authors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
