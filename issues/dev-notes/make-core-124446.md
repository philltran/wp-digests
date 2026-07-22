# Client-Side Media Processing in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Adam Silverstein
- **Published:** 2026-07-22
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/22/client-side-media-processing-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/22/client-side-media-processing-in-wordpress-7-1/)
- **Usefulness:** 5/5

## Summary

WordPress 7.1 introduces client-side media processing, shifting image compression, resizing, format conversion, rotation, and thumbnail generation from PHP to the browser using a WebAssembly build of libvips. This offloads heavy image work to the user’s device, eliminating PHP memory limits, reducing server CPU load, and delivering consistently optimized output across all hosting environments. The feature activates automatically in supporting browsers and falls back transparently to server-side processing when unsupported.

## Impact

- **Plugin developers:** `wp_generate_attachment_metadata` now fires twice per upload (`create` then `update`). Handlers must be idempotent to avoid duplicate watermarking, CDN sync, or metadata modifications.
- **Hosting & platform teams:** Server-side media hooks (`wp_image_editors`, `image_memory_limit`, `image_make_intermediate_size`) will no longer fire for client-side uploads. CSP configurations must allow `blob:` workers via `worker-src 'self' blob:;` or processing falls back to the server.
- **Theme developers:** No action required. Registered image sizes are generated client-side and deduplicated automatically.
- **Site owners & editors:** No action required. Uploads behave identically, with automatic fallback on browsers lacking `Document-Isolation-Policy` support.

## Technical details

The JS pipeline is orchestrated by `@wordpress/upload-media`, `@wordpress/vips`, `@wordpress/media-utils`, and `@wordpress/video-conversion`. PHP gating is handled by `wp_is_client_side_media_processing_enabled()`, filterable via `wp_client_side_media_processing_enabled`. When active, block editor screens emit `Document-Isolation-Policy: isolate-and-credentialless` via `wp_start_cross_origin_isolation_output_buffer()` to enable `SharedArrayBuffer`.

New REST endpoints manage the workflow: `POST /wp/v2/media/{id}/sideload` uploads generated sub-sizes, and `POST /wp/v2/media/{id}/finalize` triggers the second `wp_generate_attachment_metadata` pass with context `'update'`. The client respects the fixed `CLIENT_SIDE_SUPPORTED_MIME_TYPES` constant (`image/jpeg`, `image/png`, `image/gif`, `image/webp`, `image/avif`) and honors `wp_editor_set_quality`/`jpeg_quality` via a size-aware `image_quality` field in the upload response. Animated GIFs are converted to MP4/WebM companions recorded in `media_details.animated_video` and `media_details.animated_video_poster`. Gain-map HDR JPEGs intentionally skip `image_editor_output_format` conversion to preserve HDR data.

## Contribution

The feature was developed across multiple iterations for the WordPress 7.1 cycle, tracked in the Gutenberg repository. Core contributors include @swissspidy, @andrewserong, and @wildworks, with extensive review and documentation coordination across the Make WordPress Core team. The implementation settled on a per-document `Document-Isolation-Policy` rather than page-wide COEP/COOP headers to avoid breaking third-party embeds, and prioritized idempotent server-side hook compatibility to ensure existing watermarking and CDN plugins continue working without modification.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
