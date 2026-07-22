# #79188: Vips: inline WASM with compact UTF-8 binary encoding instead of base64

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `[Feature] Client Side Media`
- **Merged:** [`8a6ce68`](https://github.com/WordPress/gutenberg/commit/8a6ce6862b98d5d6d659cc0885b6aed80431495e)
- **Discussion:** [#79188](https://github.com/WordPress/gutenberg/pull/79188) · 4 comments · 3 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/vips` package now embeds its WebAssembly binaries using a compact UTF-8 string encoding instead of base64 data URLs. This change significantly improves compression ratios for the inlined worker bundle (`worker.min.js`), reducing gzip size by ~13% and brotli size by ~16% (~550 kB). It matters because client-side image processing (AVIF/HEIC conversion, resizing) will load faster over the network while maintaining the same inlining design that avoids separate downloads and MIME-type issues.

## Impact

- **Plugin & theme developers:** No breaking changes or API deprecations. If you fork or modify the Gutenberg build pipeline, ensure your esbuild configuration emits UTF-8 (`charset: 'utf8'`) to preserve the encoding; ASCII output will escape high bytes as `\uXXXX` and negate the compression gain.
- **Hosting & platform teams:** No action required. The change is fully contained within the built JS bundle.
- **Headless & REST consumers:** No action required. Unaffected.
- **Runtime note:** The uncompressed `worker.min.js` grows by ~16% (~1.8 MB) in memory at worker initialization, but this is transient and only impacts users who trigger client-side image processing. The compressed transfer win benefits all users.

## Technical details

- `packages/wp-build/lib/build.mjs`: The `wasmInlinePlugin` now uses a new `binaryEncode()` function that maps each byte to its UTF-8 representation (escaping only `\r`, `\n`, `\`, and quotes). The generated JS exports a `Uint8Array` decoded via a runtime loop instead of a `data:application/wasm;base64,...` string.
- `packages/vips/src/index.ts`: The `locateFile` callback in `getVips()` now passes the inlined `Uint8Array` through a new `getWasmUrl()` helper, which caches `Blob` URLs via a `WeakMap` to satisfy `wasm-vips`'s fetch-based loader.
- esbuild configuration: All relevant `bundlePackage()` and `buildWorkers()` calls now explicitly set `charset: 'utf8'` to prevent esbuild from escaping high bytes.
- Before/after pattern in `locateFile`:
  ```js
  // Before
  if ( fileName.endsWith( 'vips.wasm' ) ) {
    return VipsModule; // base64 data URL string
  }
  // After
  if ( fileName.endsWith( 'vips.wasm' ) ) {
    return getWasmUrl( VipsModule ); // Uint8Array -> Blob URL
  }
  ```

## Contribution

Opened and merged by @adamsilverstein with co-authorship from @swissspidy, incorporating the UTF-8 encoding approach originally suggested by @kleisauke. The author explicitly weighed the uncompressed memory tradeoff against the compressed transfer win before merging, confirming the tradeoff was acceptable for a lazily loaded worker.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
