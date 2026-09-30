# #82678: Fields: Resolve the MediaEdit picker through the editor.MediaUpload filter

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] Extensibility`, `[Package] E2E Tests`, `[Package] Fields`
- **Merged:** [`c69687a`](https://github.com/WordPress/gutenberg/commit/c69687a5aca5ff35ad4c53683a059e8e817b0a8f)
- **Discussion:** [#82678](https://github.com/WordPress/gutenberg/pull/82678) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The featured image field in `@wordpress/fields` now resolves its media picker through the `editor.MediaUpload` filter when rendered in the post editor (post summary / DataForm inspector), matching the behavior of the classic featured image panel and block-based pickers. Previously it used the media-utils `MediaUpload` directly, so plugins that extend the picker via `editor.MediaUpload` (Jetpack External Media, AMP, etc.) never saw the featured image field. A new `mediaUploadProps` prop on `MediaEdit` forwards `unstableFeaturedImageFlow: true` so the modal shows the "Set featured image" button and plugins can identify the context.

## Impact

- **Plugin developers extending `editor.MediaUpload`:** Your picker extensions (extra sources, validation, UI) now apply to the featured image field in the post editor's DataForm inspector. No code change needed on your side, but test your extension in that context.
- **Plugin developers extending `editor.PostFeaturedImage`:** No change to that filter's contract; the picker it receives now also goes through `editor.MediaUpload`.
- **Quick Edit (site editor pages list):** Unaffected. The plain `MediaEdit` picker is used there because `editor.MediaUpload` callbacks assume a loaded block-editor context (`editPost`, `useEntityProp`, etc.).
- **`MediaEdit` consumers (plugin/theme authors):** A new optional `mediaUploadProps` prop is available. Existing usage is unchanged; the prop is additive.
- **No action required** for sites not using the DataForm inspector experiment or not extending `editor.MediaUpload`.

## Technical details

In `packages/fields/src/components/media-edit/index.tsx`, the original single `MediaEdit` export is split into three pieces:

- `MediaEdit` (default export) — unchanged public API, uses `ConditionalMediaUpload` (the plain media-utils `MediaUpload` fallback).
- `MediaEditWithFilteredPicker` (named export, package-internal) — same body but uses `FilteredMediaUpload`, which is `withFilters( 'editor.MediaUpload' )( ConditionalMediaUpload )`.
- `MediaEditControl` — the shared implementation that accepts a `MediaUploadComponent` prop.

The `multiple` prop semantics shift in the filtered path: the block-editor `MediaUpload` API expects `'add'` (keep current selection) or `false`, so the control now passes `multiple && targetItemId === undefined ? 'add' : false` instead of the previous `multiple ? 'add' : undefined`.

In `packages/fields/src/fields/featured-image/edit.tsx`, the `FilteredMediaEdit` (wrapped in `editor.PostFeaturedImage`) now renders `MediaEditWithFilteredPicker` with `mediaUploadProps={ { unstableFeaturedImageFlow: true } }`. The non-filtered fallback (Quick Edit) renders plain `MediaEdit` with the same `mediaUploadProps` so the modal still shows the featured-image button, but without plugin extensions.

New E2E test plugin at `packages/e2e-tests/plugins/media-upload-filter/` registers a callback on `editor.MediaUpload` that appends a marker element and exposes `props.unstableFeaturedImageFlow` as a `data-featured-image-flow` attribute for assertions.

```tsx
// Before (featured image in post summary)
<MediaEdit { ...props } isExpanded />

// After
<MediaEditWithFilteredPicker
  { ...props }
  isExpanded
  mediaUploadProps={ { unstableFeaturedImageFlow: true } }
/>
```

## Contribution

Opened by @ntsekouras as part of issue #76076 (DataForm inspector back-compat). The initial version applied the `editor.MediaUpload` filter in both the post summary and Quick Edit. @ntsekouras converted the PR to draft after testing with Jetpack External Media revealed that Jetpack's Dropdown wrapper shrinks the placeholder and its source menu is hidden inside `.modal-open` in Quick Edit. @oandregal confirmed via screenshots that the Quick Edit modal is structurally different from the inspector modal. The PR was updated to scope the filter to the post summary only (mirroring the `editor.PostFeaturedImage` logic from #83133), leaving Quick Edit on the plain picker. Co-authored with @jorgefilipecosta, @andregal, and @oandregal.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
