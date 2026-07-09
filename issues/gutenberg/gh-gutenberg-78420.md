# #78420: Media: Return the filtered `wp_editor_set_quality` value in the upload response

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `[Feature] Client Side Media`
- **Merged:** [`9c9c904`](https://github.com/WordPress/gutenberg/commit/9c9c904639c2720ec831af0b42eaf20f4cbf33d6)
- **Discussion:** [#78420](https://github.com/WordPress/gutenberg/pull/78420) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds a size-aware `image_quality` field to the attachment REST API response so client-side media processing respects server-side quality filters. Previously, browser-generated sub-sizes and transcodes ignored `wp_editor_set_quality` and `jpeg_quality`, falling back to a hardcoded default. This change ensures consistent encode quality across both server-rendered and client-resized images without requiring manual filter overrides.

## Impact

- **Plugin & theme developers**: Hooks attached to `wp_editor_set_quality` and `jpeg_quality` will now govern client-generated thumbnails and transcodes, aligning browser behavior with existing server configurations.
- **Headless & REST consumers**: The `/wp/v2/media` response now includes an `image_quality` object containing a `default` quality value and optional per-sub-size overrides in `sizes`.
- **Site owners / hosting & platform**: No immediate configuration changes required; the enhancement is transparent to end users.
- **Migration/Action**: No breaking changes or code modifications needed. Existing filters automatically take effect for client-side uploads once the update ships.

## Technical details

- `lib/media/class-gutenberg-rest-attachments-controller.php`: `get_item_schema()` registers `image_quality` as a readonly object with `default` (int, 1–100) and `sizes` (object keyed by registered image sub-sizes). `prepare_item_for_response()` resolves the actual output MIME type via `image_editor_output_format`, applies `wp_editor_set_quality` and `jpeg_quality` for JPEG output, and populates the schema. A new private helper `get_image_encode_quality()` mirrors core's encode quality logic until it lands in WordPress core.
- `packages/upload-media/src/store/private-actions.ts`: `generateThumbnails()`, `resizeCropItem()`, and `getTranscodeImageOperation()` now accept a `quality` parameter (0–1 scale for the vips worker). They read `attachment.image_quality.default` and per-size overrides from `sizes`, converting the 1–100 WordPress scale to 0–1 before passing it to resize/transcode operations.
- `packages/upload-media/src/store/types.ts`: Extends the `Attachment` interface with `image_quality?: { default: number; sizes: Record<string, number> }` and adds `quality?: number` to `OperationArgs`.

## Contribution

Opened by @adamsilverstein to resolve #78419. Review feedback initially noted incomplete coverage for the legacy `jpeg_quality` filter, which was addressed before merge. The PR syncs with core development WordPress/wordpress-develop#11856 and was merged after review from @swissspidy, @westonruter, and @andrewserong.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
