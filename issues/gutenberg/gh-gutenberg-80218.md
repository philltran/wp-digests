# #80218: Client Side Media: Honor image_strip_meta and image_max_bit_depth on the client upload path

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Editor`, `[Package] Block editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`e3438eb`](https://github.com/WordPress/gutenberg/commit/e3438eb39e63bfc3ef13fa1bc7cffe62d4fc9664)
- **Discussion:** [#80218](https://github.com/WordPress/gutenberg/pull/80218) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This change ensures the `image_strip_meta` and `image_max_bit_depth` filters are honored during client-side media processing. Previously, when client-side media encoding was active, the server never instantiated `WP_Image_Editor`, causing these filters to silently fail. The fix exports the filtered values via the REST API root index, passes them through editor settings, and forwards them to the `@wordpress/vips` Web Worker, restoring parity with server-side image processing.

## Impact

- Plugin & theme developers: Hooks into `image_strip_meta` or `image_max_bit_depth` now correctly apply to images processed client-side. No code changes required.
- Site owners & platform teams: No action required. The change is transparent and maintains default behavior (metadata stripped, bit depth uncapped).
- Headless & REST consumers: The REST API root index now includes `image_strip_meta` (boolean) and `image_max_bit_depth` (integer) fields, exposed only to users with `upload_files` capability.

## Technical details

- `lib/media/load.php`: `gutenberg_media_processing_filter_rest_index()` applies `image_strip_meta` (default `true`) and `image_max_bit_depth` (default `16`) and attaches them to `$response->data`.
- `lib/compat/wordpress-7.1/preload.php`: Adds `image_strip_meta` and `image_max_bit_depth` to the `$root_fields` string for preloading.
- `packages/core-data/src/entities.js` & `packages/editor/src/components/provider/use-block-editor-settings.js`: Expose the new fields in `rootEntitiesConfig.__unstableBase` and pass them through `useBlockEditorSettings` / `useMediaUploadSettings`.
- `packages/upload-media/src/store/private-actions.ts`: `resizeCropItem` and `transcodeImageItem` read `imageStripMeta` and `imageMaxBitDepth` from `select.getSettings()` and pass them to the vips worker.
- `packages/vips/src/index.ts` & `packages/upload-media/src/store/utils/index.ts`: `convertImageFormat`, `compressImage`, and `resizeImage` accept optional `stripMeta` (boolean, defaults to `true`) and `maxBitdepth` (integer, defaults to `16`) parameters. The worker maps `stripMeta: false` to `keep: 'all'` and caps AVIF output depth at `min(source, cap)`, snapped to 8/10/12.

## Contribution

Opened and merged by @adamsilverstein, with co-authorship from @andrewserong. The PR addresses items 1 and 2 from the reconcilable hooks audit in #80216. It was backported to the `wp/7.1` branch for inclusion in WordPress 7.1. The implementation follows the existing pattern used for `big_image_size_threshold` and `image_size_threshold`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
