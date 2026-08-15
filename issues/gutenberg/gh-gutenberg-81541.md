# #81541: Media editor: keep initial modal focus on the dialog frame

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Feature] Media`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`d033d61`](https://github.com/WordPress/gutenberg/commit/d033d614e57d24a1070d1b8d6132bf88aea319d0)
- **Discussion:** [#81541](https://github.com/WordPress/gutenberg/pull/81541) · 8 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The "Edit media" modal was stealing initial focus away from its dialog frame and dropping it onto the crop area once the image finished loading. `MediaEditorCanvas` passed `focusOnMount={ focusOnMount && status === 'loaded' }` down to `Cropper`, and because that gate flips asynchronously — after `Modal` has already focused the dialog frame — focus landed mid-content on the `role="group"` "Crop area", skipping the dialog title and header actions, and could yank focus away from whatever control the user had tabbed to while the image streamed in. The fix removes the `focusOnMount` prop from `MediaEditorCanvas` entirely, so the modal keeps its default `Modal` focus behaviour and the crop area is only reachable by tabbing to it.

## Impact

**Editor users / accessibility**
- Opening the crop modal (Image block → Crop) now leaves focus on the modal dialog frame. Tab moves to the first header control instead of a resize handle, and the crop stencil no longer shows a keyboard outline on open.
- Slow-loading images no longer pull focus out from under a keyboard user mid-interaction.

**Plugin & theme developers using `@wordpress/media-editor`**
- **Prop removed:** `focusOnMount` is gone from `MediaEditorCanvasProps`. Any code rendering `<MediaEditorCanvas focusOnMount />` will fail type-checking and the prop is silently ignored at runtime. There is no replacement — remove the prop.
- `Cropper`'s own `focusOnMount` prop is **unchanged**; if you compose `Cropper` directly and want programmatic focus, keep using it there.
- The crop area retains `tabIndex={ 0 }`, so keyboard reachability and the arrow-key pan / `+`/`-` zoom interactions are unaffected.

**Site owners**
- No action required — behavioural fix only, shipping via the Gutenberg plugin with a backport targeted at WordPress 7.1.

## Technical details

Three files change in `packages/media-editor`, all in the direction of deleting the focus plumbing rather than re-timing it.

In `packages/media-editor/src/components/media-editor-canvas/index.tsx`, the `focusOnMount?: boolean` member is dropped from the exported `MediaEditorCanvasProps` interface, removed from the component signature and its JSDoc `@param` list, and the prop is no longer forwarded to `<Cropper>`:

```jsx
// Before
<Cropper
  controller={ controller }
  aspectRatio={ aspectRatio }
  freeformCrop
  focusOnMount={ focusOnMount && status === 'loaded' }
  showGrid="interactive"
  …
/>

// After
<Cropper
  controller={ controller }
  aspectRatio={ aspectRatio }
  freeformCrop
  showGrid="interactive"
  …
/>
```

The reason the old form misbehaved: `Cropper`'s focus effect keys off the `focusOnMount` prop, and `status === 'loaded'` only becomes true after the image decodes. The effect therefore fired *after* `Modal` had already applied its default `focusOnMount={ true }`, which focuses the dialog frame (`tabIndex={ -1 }`). Other Gutenberg modals either take that default or use `focusOnMount="firstContentElement"`; none focus an interior widget, so the media editor was the outlier.

In `packages/media-editor/src/components/media-editor/index.tsx`, `MediaEditorContent` stops passing `focusOnMount` when it renders `<MediaEditorCanvas>` for the image case.

The inline comment above the cropper wrapper is rewritten: the cropper still stays mounted while loading (hidden behind the spinner, `pointer-events: none` in CSS) so the image decodes off-screen and reveals in one paint, but the note now states explicitly that the crop area is never focused programmatically. A `### Bug Fixes` entry is added to the `Unreleased` section of `packages/media-editor/CHANGELOG.md` (latest tagged release: 0.16.0). Bundle impact was −22 B on `build/scripts/editor/index.min.js`.

## Contribution

The PR closes issue #81505 and was credited via props-bot to @ramonjd alongside @andrewserong, @talldan, and @himanshupathak95. The interesting part is the release logistics: @ramonjd flagged that the fix might have to be punted to 7.2 or a 7.1 minor, and @t-hamano initially said it would ride along if a 7.1 minor materialised before deciding (per a core Slack thread) to backport it to 7.1 outright. The automated cherry-pick to the `wp/7.1` branch then failed with conflicts — the bot posted the same manual-cherry-pick instructions three times — so the backport requires a hand-resolved PR based on `wp/7.1` rather than a clean pick of the trunk commit. No design disagreement about the fix itself appears in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
