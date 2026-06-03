# What’s new in Gutenberg 23.1? (07 May)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** alecgeatches
- **Published:** 2026-05-07
- **Tags:** `General`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/05/07/whats-new-in-gutenberg-23-1-07-may/](https://make.wordpress.org/core/2026/05/07/whats-new-in-gutenberg-23-1-07-may/)

## Summary

Gutenberg 23.1 introduces parallel processing for image thumbnail sideload requests to accelerate bulk uploads, repurposes the "Disable TinyMCE" experiment to conditionally hide the Classic block from the inserter, and ships two new experimental editors: Custom Taxonomies management and a Media Editor with a freeform cropper. It also expands the `@wordpress/ui` component library with `Drawer` and `Autocomplete` primitives, adds developer-preview drag-and-drop grid capabilities via `@wordpress/grid`, and resolves several real-time collaboration reliability issues tied to Yjs update bounds, offline divergence, and CRDT hydration timing.

## Impact

- **Plugin & Theme Developers**: The Classic block is now hidden from the block inserter by default. Existing Classic blocks remain fully functional. Override or restore inserter visibility using the new `wp_classic_block_supports_inserter` filter.
- **Block & UI Developers**: `@wordpress/ui` gains `Drawer` and `Autocomplete` compound primitives. Overlay components (`Dialog`, `AlertDialog`, `Drawer`) now support sticky headers/footers during vertical scroll, and all `*.Popup` overlays accept a `portal` prop with optional `*.Portal` subcomponents for customized portaling.
- **Media/Upload Handlers**: Image thumbnail generation sideloads now execute in parallel (up to the existing concurrency limit) instead of sequentially. This yields noticeable performance gains on bulk gallery uploads, large images, and slower network connections.
- **Headless & Collaboration Platforms**: RTC fixes eliminate spurious "Connection Lost" dialogs when entity counts exceed the server's per-request cap, correct size-check mismatches that previously rejected large Yjs updates, and prevent divergent states when multiple offline editors reconnect and push compactions.
- **No action required** for standard installations not utilizing the Classic block, real-time collaboration, or experimental features.

## Technical details

- **Parallel Thumbnail Sideloads**: Sideload requests for generated image thumbnail sizes previously executed sequentially. The implementation now queues them to run concurrently up to the established concurrency limit, reducing total upload finalization time (`#75888`).
- **Classic Block Inserter Control**: The `Disable TinyMCE` experiment was refocused to conditionally register the Classic block while hiding it from the inserter. Behavior is exposed via a new `wp_classic_block_supports_inserter` filter, allowing plugins or themes to force inserter visibility if needed (`#77838`, `#77840`, `#77911`, `#77845`).
- **@wordpress/ui Primitives & Overlay Polish**: Introduces `Drawer` (slide-in panels/bottom sheets) and `Autocomplete` (low-level combobox primitive). The `Dialog` component gains a `Description` subcomponent with tightened spacing/typography alignment. `Dialog`, `AlertDialog`, and `Drawer` now render sticky headers/footers during vertical overflow. All overlay `*.Popup` components accept a `portal` prop, paired with optional `*.Portal` subcomponents for tailored DOM portaling (`#76690`, `#77194`, `#77452`, `#77559`).
- **RTC Reliability & Hydration**: The "Connection Lost" dialog no longer triggers when a page registers more sync rooms than the server's per-request cap. A client/server size-check mismatch that rejected large Yjs updates was corrected. Offline divergence fixes prevent state drift when multiple users reconnect and push compactions. Sync observers are now attached only after the persisted CRDT document is hydrated, eliminating redundant block re-parsing during editor initialization (`#77631`, `#77669`, `#77980`, `#77966`).

## Contribution

Released on May 7, 2026 as part of the biweekly Gutenberg snapshot cycle. The merge spanned multiple interrelated tracks: parallelizing thumbnail sideloads (`#75888`), expanding `@wordpress/ui` with compound primitives and overlay polish (`#76690`, `#77194`, `#77452`, `#77642`), repurposing the TinyMCE experiment to suppress the Classic block in the inserter while preserving functionality (`#77838`, `#77840`, `#77911`, `#77845`), and patching real-time collaboration edge cases around Yjs size bounds, offline compaction divergence, and CRDT hydration ordering (`#77631`, `#77669`, `#77980`, `#77966`). Key contributors included first-time PR authors @adithya-naik, @danluu, @hi0001234d, @rajanarahul93, @vishnupprajapat, and @wwahammy, with continued coordination across core editor, UI, and RTC streams by maintainers @simison, @aduth, @jorgefilipecosta, and others.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
