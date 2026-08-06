# #81076: List View: Only subscribe to the block subtree for rows that can show images

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Feature] List View`, `[Package] Block editor`
- **Merged:** [`ff05a8c`](https://github.com/WordPress/gutenberg/commit/ff05a8ca62ee8857f9075dc49e2bd0d544bec55b)
- **Discussion:** [#81076](https://github.com/WordPress/gutenberg/pull/81076) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Block Editor's List View now avoids subscribing to the full block subtree for rows that cannot display image previews. By gating the `getBlock` call inside the `useListViewImages` hook on a whitelist of image-capable block types, the editor eliminates unnecessary re-renders when scrolling or expanding list items. This reduces UI lag and improves List View performance in documents containing many non-image blocks.

## Impact

- **Block editor users**: Smoother List View navigation and fewer unnecessary re-renders when interacting with large documents.
- **Plugin & theme developers**: No API changes, deprecations, or migration steps required.
- **Hosting & platform teams**: No server-side impact; this is a client-side JavaScript optimization.
- **No action required** for existing sites or custom block development.

## Technical details

The change modifies `packages/block-editor/src/components/list-view/use-list-view-images.js`. Previously, `useListViewImages` unconditionally called `select( blockEditorStore ).getBlock( clientId )` inside `useSelect`, which subscribes to the entire block subtree and triggers re-renders on any nested block change. The diff now reads the block name first via `getBlockName( clientId )`, checks it against the `IMAGE_GETTERS` object (which maps `core/image`, `core/cover`, `core/media-text`, and `core/gallery`), and only calls `getBlock( clientId )` if a match exists. A minor cleanup also inlines the `getImages` lookup in `getImagesFromBlock`.

Before/after usage pattern in the hook:
```js
// Before
return { block: select( blockEditorStore ).getBlock( clientId ) };

// After
const { getBlockName, getBlock } = select( blockEditorStore );
const hasImages = !! IMAGE_GETTERS[ getBlockName( clientId ) ];
return { block: hasImages ? getBlock( clientId ) : undefined };
```

## Contribution

Opened and merged by @Mamaduka with co-authorship from @andrewserong. The change was reviewed to confirm that reading a block object subscribes to its entire subtree, prompting the conditional gate. The upstream record contains no additional design debate or alternative approaches.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
