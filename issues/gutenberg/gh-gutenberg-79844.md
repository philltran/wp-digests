# #79844: Media Modals: Invalidate attachment caches when closing the modal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Media Utils`
- **Merged:** [`f9676dc`](https://github.com/WordPress/gutenberg/commit/f9676dc53e6d05f1d7323deb2f791fb074d68e45)
- **Discussion:** [#79844](https://github.com/WordPress/gutenberg/pull/79844) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

When media is uploaded through the legacy wp.media modal, newly attached assets fail to appear in editor components relying on cached queries (e.g., the Dynamic Gallery block). This change extracts the attachment cache invalidation logic into a shared utility and calls it when the modal closes, ensuring getEntityRecords resolutions for postType/attachment are refreshed without requiring a full editor reload.

## Impact

- **Block & Editor Developers**: No code changes required. The MediaUpload component now automatically invalidates stale attachment queries on close, resolving gallery and media-list placeholders that previously failed to update after uploads.
- **Plugin/Theme Developers**: Extensions using the core wp.media modal or @wordpress/media-utils MediaUpload component inherit the fix automatically; no migration is needed.
- **No action required** for site owners, REST API consumers, or non-editor-facing extensions.

## Technical details

- Introduces `packages/media-utils/src/utils/invalidate-attachment-resolutions.ts`, exporting `invalidateAttachmentResolutions(registry)`. This function iterates `registry.select(coreStore).getCachedResolvers()`, isolating the getEntityRecords cache, and calls `invalidateResolution('getEntityRecords', args)` exclusively for entries where `args[0] === 'postType' && args[1] === 'attachment'`.
- Refactors `packages/media-utils/src/components/media-upload-modal/use-invalidate-attachment-resolutions.ts` to delegate to the new utility instead of inlining the resolution walk.
- Patches the MediaUpload class in `packages/media-utils/src/components/media-upload/index.js`, injecting a call to `invalidateAttachmentResolutions({ select, dispatch })` in the close handler before `this.frame.detach()`. This aligns the legacy modal's behavior with the experimental DataViews-powered modal, which already performed this invalidation post-upload.
- Unit tests verify selective invalidation (only attachment queries are touched) and ensure the helper runs before frame detachment.

## Contribution

Opened by @andrewserong to address stale gallery updates observed during Dynamic Gallery block testing (#79844). Co-authored by @tyxla, @talldanwp, and @ramonJD after the team consolidated split logic between legacy and experimental modal implementations into a single shared utility. Merged as f9676dc; follow-up invalidation for media categories is deferred to #79921.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
