# #78758: Media Editor: Remove resize handles toggle from crop panel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`8a824b3`](https://github.com/WordPress/gutenberg/commit/8a824b3febe9d8e692ca4957fd531a6bf2431dc2)
- **Discussion:** [#78758](https://github.com/WordPress/gutenberg/pull/78758) · 5 comments · 1 reactions

## Summary

This PR removes the user-facing **"Show resize handles"** toggle from the media editor's Crop panel in the Gutenberg media modal. The toggle exposed an implementation detail (the cropper's `freeformCrop` behavior) rather than a clear user goal, and disabling it removed the primary affordance for resizing a free-form crop. The media modal now always enables crop resize handles, leaving only `Aspect ratio` and `Zoom` controls in the Crop panel. The lower-level cropper `freeformCrop` prop is unchanged and remains available for internal/custom cropper integrations.

## Impact

- **Editor users:** The Crop panel no longer shows a "Show resize handles" toggle. Resize handles are now always visible on the crop box by default — for both free-form and fixed-ratio (e.g. Square) crops. No setting to learn or restore.
- **Plugin & theme developers:** This is a UI-only change in the `@wordpress/media-editor` package. The cropper's `freeformCrop` prop is **not** removed — it stays available for custom cropper use. No public API is removed or deprecated.
- **No action required** for sites or integrations. If you embed the lower-level cropper directly and relied on `freeformCrop`, that prop still works.

## Technical details

The change is scoped to the media editor's crop UI under `packages/media-editor/src/components/media-editor-canvas/` (and the associated crop panel component). The control that previously toggled the cropper's `freeformCrop` behavior is removed from the panel's rendered controls; the media modal now passes resize handles as always-on rather than gating them behind a user preference.

Before (conceptual):

```
Crop panel:
  - Aspect ratio
  - Zoom
  - Show resize handles  (toggle → cropper freeformCrop)
```

After:

```
Crop panel:
  - Aspect ratio
  - Zoom
  (resize handles always enabled)
```

Behavior is preserved across aspect-ratio modes: handles drag freely when ratio is `Free`, and constrain to the selected ratio (e.g. `Square`) otherwise; Reset returns options to defaults with handles still visible. The underlying cropper `freeformCrop` prop is retained for internal/custom use. The build impact was a net reduction of 148 B in `build/scripts/editor/index.min.js`.

## Contribution

Opened and merged by **@ramonjd** (commit `8a824b3`), with **@andrewserong** credited as co-author. The removal was prompted by **@jasmussen**, who noted the toggle exposed an implementation detail rather than a user goal; he added that it would be "easy to add back if someone unearths a really strong use case for it." **@cbravobernal** cherry-picked the change to the `release/23.3` branch for inclusion in the next release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
