# #76707: Fix image upload crashes  

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Status] In Progress`, `[Package] Editor`, `[Package] Block editor`, `[Feature] Client Side Media`
- **Merged:** [`ec86317`](https://github.com/WordPress/gutenberg/commit/ec8631726ad7682d8fe7930b806519b73bd30b4a)
- **Discussion:** [#76707](https://github.com/WordPress/gutenberg/pull/76707) · 6 comments · 2 reactions

## Summary

Fixes client-side media processing crashes caused by unbounded libvips WASM memory growth and stuck upload spinners when handling unsupported image formats. The change disables the libvips operation cache, implements worker recycling after 50 operations to reclaim linear memory, and corrects parent-child synchronization during upload cancellation. These fixes prevent out-of-memory errors and UI hangs for editors performing bulk or repeated media uploads.

## Impact

- **Plugin & theme developers**: No breaking changes or new public APIs. Existing client-side upload handlers remain compatible. Ensure your `onUploadError` logic accounts for cancelled child operations to avoid stale UI states during batch processing.
- **Hosting & platform teams**: No configuration changes required. The reduced WASM memory footprint prevents OOM crashes in constrained environments where the block editor processes media client-side.
- **Site owners/editors**: Bulk uploads and unsupported format handling no longer freeze the interface or crash the process. Error states surface correctly instead of leaving persistent spinners.
- **No action required** for existing editors and plugins.

## Technical details

- Modifies the client-side upload pipeline to manage libvips worker lifecycle and concurrency controls.
- Sets `Cache.max(0)` to disable libvips operation caching, directly preventing unbounded WASM memory growth that caused OOM crashes.
- Implements a hard limit for worker recycling: completed vips operations are tracked, and the worker is terminated/recreated after 50 operations to reclaim linear memory (which cannot shrink in WASM).
- Fixes `cancelItem` behavior in the upload queue: when a parent item is cancelled, pending child operations waiting on concurrency slots are now unblocked and processed. The parent's Finalize gate is also triggered immediately upon cancellation, resolving stuck spinner states.
- Redirects wasm-vips internal `stdout`/`stderr` to suppress noisy console output during AVIF encoding/decoding; debug output remains accessible via `globalThis.__vipsDebug`.
- Adds performance benchmarks in `test/performance/specs/media-upload.spec.js` covering JPEG, PNG, large JPEG, and batch (5-image) uploads.

## Contribution

Opened and merged as part of addressing #76706 by @adamsilverstein, with co-authors @andrewserong and @swissspidy. The PR directly resolved OOM crashes and queue synchronization bugs reported during client-side media processing. A known visual fallback issue for failed AVIF uploads was acknowledged but did not block the merge; the core stability fixes were accepted and landed in commit `ec86317`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
