# #78796: Dynamic Gallery Block: Attempt a dynamic mode of the gallery block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Package] Block library`, `[Block] Gallery`, `[Type] Experimental`
- **Merged:** [`2212121`](https://github.com/WordPress/gutenberg/commit/221212183b3682d5cd81da63258a2e78f281548a)
- **Discussion:** [#78796](https://github.com/WordPress/gutenberg/pull/78796) · 16 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds an experimental dynamic mode to the `core/gallery` block, enabling it to render media items attached to the current post rather than relying exclusively on static inner blocks. This introduces a `dynamicContent` attribute to store query arguments and ordering preferences, while providing an editor preview pipeline and server-side rendering that mirrors classic shortcode behavior.

## Impact

- **Block & theme developers**: The Gallery block now consumes `postId` and `postType` via `usesContext` and accepts a new `dynamicContent` object attribute. Extenders must ensure custom render logic or context consumers do not conflict with dynamic state, as rendering pipelines and inner-block mounting behavior shift in this mode.
- **Site owners & content editors**: A new "Source" panel appears in the Inspector Controls, allowing toggling between static and dynamic galleries. Switching to dynamic discards manually added images behind a confirmation dialog. No immediate code changes required unless custom themes or plugins heavily target Gallery block markup structure.
- **Platform/Hosting**: No server-side dependency changes beyond standard media REST endpoints; dynamic queries are resolved client-side during editor preview and on the frontend via existing attachment metadata.

## Technical details

- Updated `packages/block-library/src/gallery/block.json` to add `postId` and `postType` to `usesContext` and introduce the `dynamicContent` attribute (type: object).
- Added `packages/block-library/src/gallery/dynamic-source.js`, which defines the `ATTACHED_MEDIA = 'core/attached-media'` discriminator, default ordering (`date/desc`), and a descriptor map for UI strings.
- Added `packages/block-library/src/gallery/dynamic-gallery.js` containing `GallerySourcePanel` (Inspector Controls for mode switching and order configuration), `GalleryImagesPreview` (renders read-only previews via `useBlockPreview` with `display: contents` to keep captions editable), and `GalleryDynamicView` (handles canvas rendering, empty-state placeholders, and inner block mounting to suppress List View).
- Behavior enforces a confirmation dialog when transitioning from static to dynamic mode (`requestEnableDynamicMode`), guards against structural toolbar exposure during resolution (`isResolvingDynamic` disabled state), and intentionally omits `menu_order` support due to REST API constraints.

## Contribution

Open by @andrewserong as an experimental MVP to bridge classic gallery shortcode behavior into the block editor, merged via commit `2212121` with co-authors @tyxla, @ramonjd, and @fcoveram. Iterative review focused on UI/UX flows for mode switching, implementing a read-only preview that respects gallery context, and explicitly excluding manual media order due to endpoint limitations. Discussion refined terminology around "attached to the post" and tightened content-lock guards before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
