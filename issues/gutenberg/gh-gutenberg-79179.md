# #79179: Vips: bump wasm-vips to 0.0.18 for high-bit-depth AVIF decoding

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Client Side Media`
- **Merged:** [`bed156d`](https://github.com/WordPress/gutenberg/commit/bed156d66de7d81a6dc6671b079b6262f56ba82a)
- **Discussion:** [#79179](https://github.com/WordPress/gutenberg/pull/79179) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/vips` package bumps `wasm-vips` from `0.0.17` to `0.0.18`, enabling native client-side decoding of 10- and 12-bit (high-bit-depth) AVIF images. This resolves a bug where client-side media processing would fail with tile decode errors on HDR AVIF uploads. The update also eliminates the need for a previously vendored custom WASM binary and multi-threaded worker setup, as the upstream release handles high-bit-depth decoding natively in a single-threaded build.

## Impact

- **Plugin & theme developers / Client-side media consumers:** No code changes required. The `@wordpress/vips` API remains identical, but uploads of 10/12-bit AVIFs will now process successfully instead of failing.
- **Hosting & platform teams:** The `worker.min.js` bundle size decreases by ~547 kB (−6.36%), improving load times for client-side media processing workflows.
- **No action required** for existing integrations using `@wordpress/vips`.

## Technical details

The diff updates `packages/vips/package.json` and `package-lock.json` to resolve `wasm-vips@^0.0.18`. Because the upstream package no longer hoists to the workspace root, `test/e2e/bin/gen-ultrahdr-fixture.mjs` was modified to resolve the module via `createRequire` pointing to `packages/vips/node_modules/wasm-vips` instead of using a direct `import`. The package continues to ship `vips.wasm` and `vips-heif.wasm` (inlined as base64 data URLs by `wasmInlinePlugin` in `packages/wp-build/lib/build.mjs`), but the upstream build now links `libaom` with `CONFIG_AV1_HIGHBITDEPTH=1`. An integration test (`packages/vips/src/test/highbitdepth-avif.ts`) was added to verify decoding without mocking, confirming that 10/12-bit AVIFs decode to a `ushort` (16-bit) format while 8-bit AVIFs decode to `uchar`.

## Contribution

Opened to address a regression in client-side HDR AVIF handling, the PR was merged after brief review. The author explicitly rejected a prior approach that involved vendoring a custom WASM binary and implementing a multi-threaded Web Worker model, opting instead for the upstream release which natively supports the feature. During review, a WASM binary encoding optimization was validated but deferred to a follow-up PR to keep this change scoped to the dependency bump.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
