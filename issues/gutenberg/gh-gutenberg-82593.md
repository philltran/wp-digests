# #82593: Media & Text Block: Add a Media section to the block inspector panel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Media & Text`, `[Package] Block editor`
- **Merged:** [`565f178`](https://github.com/WordPress/gutenberg/commit/565f1788e9d318d6918eacbdc4ef2f0af5b13101)
- **Discussion:** [#82593](https://github.com/WordPress/gutenberg/pull/82593) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Media & Text block now has a **Media** section in the block inspector (Content tab) for setting, replacing, and resetting media from the sidebar, not just the toolbar. It reuses the `MediaControl` component already used by Image and Site Logo, shows a thumbnail and filename, reads "Add media" when empty, and includes the "Use featured image" option. The section renders only when the block is selected.

## Impact

- **Site owners / editors:** Media & Text media can now be added, replaced, and reset from the inspector. The toolbar behavior is unchanged.
- **Plugin & theme developers:** No breaking changes or deprecations. `MediaControl` in `block-library/src/utils/media-control.jsx` gains two optional props, `useFeaturedImage` and `onToggleFeaturedImage`, which are passed through to `MediaReplaceFlow`. This is an internal block-library util, not a public package API.
- **Behavior change in `MediaReplaceFlow`:** choosing "Use featured image" now also closes the dropdown. This applies to every consumer that passes `onToggleFeaturedImage`.
- **Headless / REST / hosting:** Not affected. There are no block.json, attribute, or serialization changes.
- No action required.

## Technical details

**`media-text/edit.jsx`:** adds a `mediaInspectorPanel` element, rendered only when `isSelected`. It is an `<InspectorControls group="content">` wrapping a `ToolsPanel` labeled "Media" with a single `ToolsPanelItem` (`isShownByDefault`). `hasValue` is `!! mediaUrl || !! useFeaturedImage`. Both `onDeselect` and `resetAll` call `onSelectMedia( undefined )`.

The item renders `MediaControl` with:
- `mediaUrl={ mediaUrl || featuredImageURL }`
- `filename` resolved as `image?.media_details?.sizes?.full?.file || image?.slug || getFilename( mediaUrl )`
- `allowedTypes={ ALLOWED_MEDIA_TYPES }`
- `onSelect={ onSelectMedia }`
- `onReset={ () => onSelectMedia( undefined ) }`
- `useFeaturedImage` and `onToggleFeaturedImage={ toggleUseFeaturedImage }`
- `emptyLabel` set to "Add media"

Upload errors are routed through `createErrorNotice( message, { type: 'snackbar' } )` from `noticesStore`.

**`media-text/constants.js`:** `ALLOWED_MEDIA_TYPES = [ 'image', 'video' ]` is now exported. `media-container.jsx` imports it instead of defining it locally.

**`utils/media-control.jsx`:** `MediaControl` accepts `useFeaturedImage` and `onToggleFeaturedImage` and forwards them to `MediaReplaceFlow`. The JSDoc is updated.

**`block-editor/.../media-replace-flow/index.jsx`:** the "Use featured image" `MenuItem` `onClick` changes as follows.

```jsx
// before
onClick={ onToggleFeaturedImage }
// after
onClick={ () => {
	onToggleFeaturedImage();
	onClose();
} }
```

## Contribution

During review, @talldan raised how the new controls behave inside an unsynced pattern, where every Media & Text and Image block would show its own media picker. The fix was to render the section only when the block is selected, matching the Image block. @jasmussen questioned how this scales with several images in a pattern and pointed to an older design for a Content panel that lists blocks with icons and labels. He suggested that direction for pattern editing, and it was not adopted here. @talldan judged the selected-block-only approach a reasonable initial decision, noting the Content panel already takes up a lot of space. The PR was motivated by issue #66563 and by the block fields experiment's removal in #82531, and it is related to #82751, which does the same for Image. The author disclosed using Claude Code.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
