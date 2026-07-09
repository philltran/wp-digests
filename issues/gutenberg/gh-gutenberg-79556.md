# #79556: Vips: preserve bit depth of high-bit-depth AVIF in sub-sizes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Status] In Progress`, `[Feature] Client Side Media`
- **Merged:** [`fa55aa0`](https://github.com/WordPress/gutenberg/commit/fa55aa025e1220ab96fab1d63cbb79347e2d7da1)
- **Discussion:** [#79556](https://github.com/WordPress/gutenberg/pull/79556) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Vips client-side media package now preserves the original 10/12-bit depth of high-bit-depth AVIF images during resize operations, preventing silent flattening to 8-bit sRGB in generated sub-sizes. Previously, decoding succeeded but thumbnail processing and missing save options caused precision loss or bit-depth inflation. This fix aligns sub-size output with the source bit depth for HDR content without altering existing 8-bit or animated workflows.

## Impact

- **Site owners & platform admins:** High-bit-depth AVIF uploads will now generate correctly preserved bit depths across all resized thumbnails, maintaining HDR fidelity.
- **Plugin & theme developers:** No breaking API changes. The internal resize pipeline automatically routes high-bit-depth stills through a precision-preserving path. No code migration required, but media handling behavior will shift for HDR sources.
- **Headless & REST consumers:** N/A (client-side WASM-Vips processing only; server-side WP_Image_Editor workflows are unaffected).

## Technical details

- **File:** packages/vips/src/index.ts
- **Detection:** Added getSourceBitdepth() to read the heif-bitdepth metadata field from decoded images via image.getInt( 'heif-bitdepth' ), returning 10 or 12 for HDR sources and defaulting to 8.
- **Resize Path Bypass:** For high-bit-depth still AVIFs, the pipeline now bypasses thumbnailBuffer (which performs a color-managed export flattening samples to 8-bit sRGB uchar). Instead, it uses a new resizeHighBitDepth() function that replicates thumbnail geometry using direct resize() and crop() operations on the full 16-bit ushort image.
- **Save Options:** Updated buildSaveOptions(), compressImage(), and convertImageFormat() to accept a bitdepth parameter. Explicitly passing this to heifsave prevents default inflation (e.g., 10-bit inflating to 12-bit) and ensures precise round-trips like 10→10 or 12→12.
- **Type Updates:** SaveOptions interface in packages/vips/src/types.ts now explicitly documents the bitdepth?: number field.

## Contribution

Opened by @adamsilverstein as a follow-up to #79179, with code review and scoping feedback from @gregbenz and @kleisauke. The PR was merged after passing both mocked unit tests (verifying routing logic) and real WASM-Vips integration tests (verifying actual bit-depth preservation). HDR color metadata preservation (NCLX/CLLI) was explicitly scoped out as an upstream libvips dependency and tracked separately in #79559.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
