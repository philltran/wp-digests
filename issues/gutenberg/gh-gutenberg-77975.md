# #77975: Vips: Remove dead batchResizeImage and vipsBatchResizeImage exports

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Status] In Progress`, `[Feature] Client Side Media`
- **Merged:** [`9eeeebf`](https://github.com/WordPress/gutenberg/commit/9eeeebf5debf0a7c5bf2301ea5c63204e0afb5bd)
- **Discussion:** [#77975](https://github.com/WordPress/gutenberg/pull/77975) · 3 comments · 0 reactions

## Summary

The `@wordpress/vips` package removes its dead batch resizing API surface, including the WASM-side `batchResizeImage()` function and the main-thread RPC wrapper `vipsBatchResizeImage()`. This cleanup eliminates orphaned worker code that was fully superseded by a per-sub-size concurrent sideload architecture. No functional behavior changes occur for consumers; the shared resize helpers used by the active `resizeImage()` path remain intact.

## Impact

- **Plugin & Theme Developers:** No action required. The removed symbols were internal to the Gutenberg build and had zero external callers.
- **Gutenberg/Core Contributors:** The public API surface of `@wordpress/vips` shrinks slightly (two exported functions and two TypeScript interfaces removed). If you vendored or forked recent builds, strip references to `batchResizeImage`, `vipsBatchResizeImage`, `BatchResizeConfig`, and `BatchResizeResult`.
- **Headless & REST Consumers:** No impact. VIPS operates exclusively in the client-side editor context.

## Technical details

- **Removed Exports:**
  - WASM-side worker function: `batchResizeImage()`
  - Main-thread RPC wrapper: `vipsBatchResizeImage()`
  - Worker API surface entry: `batchResizeImage`
  - TypeScript interfaces: `BatchResizeConfig`, `BatchResizeResult`
- **Retained Code:** Shared helpers `applyResizeAndCrop` and `buildSaveOptions` in `packages/vips/` are preserved and still used by the active `resizeImage()` path.
- **Architectural Context:** The upload pipeline in `packages/upload-media/src/store/private-actions.ts` no longer imports or calls batch functions. Parallelism is now handled by queuing individual `ResizeCrop` + `Upload` sideload items per thumbnail, following the reorganization in #75888.
- **Build Impact:** `-217 B` (0%) reduction in `build/modules/vips/worker.min.js`. Unit test suite for `upload-media` and `vips` passes cleanly.

## Contribution

Opened and merged by @adamsilverstein following investigation into #77247, which questioned the viability of the batch resize fallback. Code review confirmed the batch path had been completely superseded by concurrent sideload dispatch, leaving zero callers in `upload-media`. Co-authored with @andrewserong, the PR was merged as a straightforward dead-code removal after passing unit tests and lint checks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
