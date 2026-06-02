# #78711: Image Editor: focus return after closing image crop modal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Package] Editor`, `[Package] Block library`
- **Merged:** [`898fda7`](https://github.com/WordPress/gutenberg/commit/898fda7c463b4fccf3ce8bb7a728b357b92faddf)
- **Discussion:** [#78711](https://github.com/WordPress/gutenberg/pull/78711) · 9 comments · 0 reactions

## Summary

This fixes a keyboard accessibility bug in the block editor's image cropper. When the media editor modal is opened from a block toolbar's Crop button via keyboard, closing it (via Cancel, Escape, or Save) left focus lost rather than returning it to the launching control. The PR adds an optional `onClose` callback to the media editor modal's open payload and wires up a focus-return callback from the Image block (including within Gallery) and the Site Logo block, so focus returns to the Crop toolbar button after the modal closes.

## Impact

- **Site owners / editors:** Keyboard and screen-reader users regain expected focus behavior — after cropping (or cancelling) an image from the Image, Gallery, or Site Logo block, focus returns to the Crop toolbar button instead of being dropped to `document.body`.
- **Plugin & theme developers:** The media editor modal open payload now accepts an optional `onClose` callback, invoked when the modal closes (including after save). If you open the media editor programmatically, you can pass `onClose` to restore focus or run cleanup. Purely additive — **no action required** for existing integrations.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The change threads an optional `onClose` callback through the media editor modal's open flow and invokes it on close, including the post-save close path. The Image and Site Logo blocks pass a focus-return callback that targets the toolbar Crop button via a ref:

```js
// Image / Site Logo block, when opening the cropper:
openMediaEditor( {
  // ...existing payload
  onClose: () => cropButtonRef.current?.focus(),
} );
```

The underlying problem (surfaced during E2E debugging) is a focus-restoration race: on close, the focused Crop area inside the modal unmounts first, the browser drops focus to `document.body`, and then `useFocusReturn()` cleans up the Modal frame — but because the frame is still connected and focus is already outside it, the hook exits early (see `packages/compose/src/hooks/use-focus-return/index.js`) and never restores focus to the launcher. The explicit `onClose` focus callback sidesteps this by deterministically refocusing the originating Crop button. Build-size deltas land in `block-library` (+55 B), `editor` (+41 B), and `block-editor` (-41 B), reflecting the changes across the Image/Site Logo blocks and the editor's media editor modal handling.

## Contribution

Authored by **@ramonjd** and co-authored by **@andrewserong**, landing in Gutenberg as an enhancement labeled Accessibility (a11y), merged in commit `898fda7`. During review, a flaky E2E failure prompted investigation; @ramonjd traced the focus loss to `useFocusReturn()` bailing out early after the in-modal Crop area unmounted and focus fell back to `document.body`, confirming the explicit `onClose` focus-return callback as the fix rather than relying on the modal's built-in focus restoration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
