# #74873: Media: Add UltraHDR (ISO 21496-1) gain map support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `[Feature] Client Side Media`
- **Merged:** [`2d81374`](https://github.com/WordPress/gutenberg/commit/2d8137413db62d5533891872bab5c62df7888b69)
- **Discussion:** [#74873](https://github.com/WordPress/gutenberg/pull/74873) · 34 comments · 1 reactions

## Summary

Adds client-side detection and preservation of ISO 21496-1 UltraHDR gain maps to the WordPress media upload workflow. When an UltraHDR JPEG is uploaded, the system probes the file via the `@wordpress/vips` WebAssembly worker, uploads the original unmodified, and ensures all generated sub-sizes retain the embedded gain map. This enables automatic HDR rendering on compatible displays without stripping the auxiliary image data during client-side scaling.

## Impact

- **Plugin & theme developers**: No breaking changes to REST APIs or PHP handlers. The upload queue now routes UltraHDR JPEGs through a new client-side resize path that skips format transcoding to preserve the gain map.
- **Headless & REST consumers**: Unaffected at the JSON level; images remain `image/jpeg`. SDR displays will continue to render the embedded base image automatically as per ISO 21496-1 specs.
- **Action required**: None for existing integrations. Ensure custom upload flows that bypass `@wordpress/upload-media` account for the new client-side processing behavior if they interact with generated sub-sizes.

## Technical details

- Bumps `wasm-vips` from `^0.0.16` to `^0.0.17`, which provides native `uhdrload`/`uhdrsave` support via Google's `libultrahdr`.
- Introduces `OperationType.DetectUltraHdr` and a `detectUltraHdr()` thunk in `packages/upload-media/src/store/private-actions.ts`. The thunk reads the file as an `ArrayBuffer`, calls `vipsGetUltraHdrInfo(buffer)` from `@wordpress/vips` utils, and gracefully handles probe failures.
- Adds a module-level `ultraHdrItems: Set<QueueItemId>` to track parent items that successfully probe as UltraHDR. This set is cleared in `removeItem()` to prevent unbounded memory growth.
- Modifies `generateThumbnails()` to check `ultraHdrItems.has(item.id)`. When true, it skips `TranscodeImage` operations (`!isUltraHdr && outputMimeType` guard) and routes sub-sizes through a gain-map-aware resize step that downsizes the base image and re-embeds the ISO 21496-1 map in lockstep.
- Removes legacy `open-ultrahdr`/`open-ultrahdr-wasm` dependencies and the standalone `EncodeUltraHdr` queue operation, consolidating all processing through the single vips worker.

## Contribution

Merged by @adamsilverstein following iterative review of the client-side media upload pipeline. Key upstream work was contributed by @kleisauke, who landed native UltraHDR support in `wasm-vips`. The discussion centered on probe performance and transcoding strategy; reviewers agreed to default to simple gain map preservation without adding complex compression tuning or aggressive SDR extraction, deferring those options to future iterations. The PR closed #74874 and aligned with ongoing client-side media experiments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
