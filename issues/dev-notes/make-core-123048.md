# Call for Testing: client-side media processing

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Adam Silverstein
- **Published:** 2026-06-04
- **Tags:** `Core`, `call for testing`, `media`
- **Link:** [https://make.wordpress.org/core/2026/06/04/call-for-testing-client-side-media-processing/](https://make.wordpress.org/core/2026/06/04/call-for-testing-client-side-media-processing/)

## Summary

Client-side media processing is graduating from a Gutenberg experiment to a progressive enhancement for WordPress 7.1. The block editor now decodes, resizes, and encodes uploaded images locally via a VIPS WebAssembly pipeline (`wasm-vips`) before transmitting them to the server. This offloads CPU/memory work, enables uniform support for modern formats (AVIF, WebP, HEIC, UltraHDR, JPEG XL), and silently falls back to the existing server-side pipeline for incompatible browsers or low-memory devices.

## Impact

['- **Plugin & theme developers**: Media optimizers, DAM integrations, and custom workflows should be validated against the new client-side upload path. The `image_editor_output_format` filter and extensibility hooks/actions from PR #74913 enable output configuration, but developers must account for pre-encoded client payloads and newly supported transcodes (e.g., HEIC→WebP/AVIF, GIF→MP4/WebM).', '- **Hosting & platform**: Server-side image processing load decreases during uploads. However, CSP configurations blocking `worker-src` for `blob:` URLs will trigger silent server-side fallbacks without user-visible errors.', '- **Headless & REST consumers**: No breaking changes to existing media endpoints; upload payloads may now contain client-generated sub-sizes depending on capability detection.', '- **No action required** for standard installations; browser and device capability checks automatically gate activation.']

## Technical details

['- Activation is gated by `window.__clientSideMediaProcessing`, which returns `true` when the WASM pipeline is ready and constraints are met (≥2 GB RAM, excludes `2g`/`slow-2g`/`Save-Data` headers).', '- The editor intercepts upload blobs, decodes/resizes/encodes them via an on-demand `wasm-vips` worker, and transmits pre-sized payloads to the sideload endpoint. PHP server-side processing acts as a fallback when Document-Isolation-Policy is unsupported (non-Chromium browsers), CSP blocks `worker-src`, or memory thresholds fail.', '- Extensibility exposes PR #74913’s filters/actions and the `image_editor_output_format` filter to dictate output formats (e.g., forcing AVIF/WEBP from a JPEG source). HEIC canvas fallbacks (#76731) and gain map preservation handle modern formats.', '```diff\n- // Prior server-side-only flow: editor sends original blob, PHP wp_image_editor resizes on upload response\n+ // New client-side progressive flow: browser decodes/resizes via wasm-vips, transmits pre-encoded sub-sizes to sideload endpoint\n```']

## Contribution

Proposed by Adam Silverstein as a WordPress 7.1 progression building on the closed 7.0 iteration (#74333). The current test cycle (tracked in #76756) focuses on expanding format support, optimizing bundle weight, and hardening upload resilience. Key technical contributions include the extensibility API (PR #74913), HEIC canvas fallbacks (#76731), GIF-to-video conversion (#76946), CSP-safe worker handling (#76765), and save-during-upload race condition guards (#76973). No final merge date is set; the post coordinates community validation ahead of the 7.1 release window.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
