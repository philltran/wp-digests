# #81431: Fix MediaPlaceholder drag eligibility

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`f747623`](https://github.com/WordPress/gutenberg/commit/f74762325c64a934ae8843bb0eaedb3edbb42bf6)
- **Discussion:** [#81431](https://github.com/WordPress/gutenberg/pull/81431) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`MediaPlaceholder`'s drop-zone eligibility check no longer passes vacuously for canvas block-reorder drags, which carry only the bare `wp-blocks` transfer type. The PR also attaches the `useInnerBlocksProps()` drop-zone props to the Playlist block's track `<ol>`. Together these let Playlist Track blocks be reordered with before/after insertion markers, without a media drop zone appearing over them.

## Impact

- **Plugin & theme developers:** Any block that renders `MediaPlaceholder` (with `multiple` or single mode) and contains draggable inner blocks was affected. The media drop zone no longer activates for canvas block drags. It still activates for inserter drags of allowed media blocks and for file drops. No API changes; no action required.
- **Site owners / editors:** Reordering Playlist Track blocks now works and shows insertion markers.
- **Custom blocks mimicking inserter drags:** Eligibility now requires at least one `wp-block:core/*` type in `dataTransfer.types`. A custom drag source that previously relied on an empty type list being accepted would no longer match.
- **Headless / REST / hosting:** Not affected.

## Technical details

**`packages/block-editor/src/components/media-placeholder/index.js`:** the `isEligible` callback on the `DropZone` collects `dataTransfer.types` entries prefixed with `wp-block:core/` into `types`. Previously it returned `types.every( allowed ) && ( multiple ? true : types.length === 1 )`. For a canvas reorder drag, `types` was `[]`, so `[].every()` returned `true` and with `multiple` the zone activated. Now `types.length > 0 &&` is prepended, so the empty array is rejected. A comment documents that only the inserter publishes dragged block names as `wp-block:core/{name}`.

**`packages/block-library/src/playlist/edit.js`:** `useInnerBlocksProps` was previously called with `blockProps` and only its `children` was used, while the `<ol>` rendered separately with a hand-built `className`. It is now called with the tracklist `className` (the `clsx` expression moved up) and the result is split:

```js
const { children: innerBlocks, ...trackListProps } = useInnerBlocksProps(
	{ className: clsx( 'wp-block-playlist__tracklist', { /* ... */ } ) },
	{ __experimentalAppenderTagName: 'li', renderAppender: false }
);
// ...
<ol { ...trackListProps }>
	<PlaylistContext.Provider value={ playlistContext }>
		{ innerBlocks }
	</PlaylistContext.Provider>
</ol>
```

This marks the `<ol>` as the inner-blocks container so the editor's drop-zone/insertion-marker logic targets it. CHANGELOG entries were added for `block-editor` and `block-library`. No hooks, REST schema, or `block.json` changes.

The PR discussion notes that a broader approach, parsing and rejecting `type: 'block'` in `MediaPlaceholder`, was deliberately not taken.

## Contribution

@ellatrix's review found that after the placeholder fix alone, tracks still could not be moved and no indicator appeared between them. @scruffian then added the Playlist `useInnerBlocksProps` change as a second fix. He chose the narrow `types.length > 0` guard over making `MediaPlaceholder` parse and reject block drags, since `dataTransfer.getData()` isn't reliably available during drag-over in every browser, which is why the type-list convention exists. Broader block rejection is deferred to #81794. @jeryj is credited in the props list. The PR notes OpenAI Codex assisted with the changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
