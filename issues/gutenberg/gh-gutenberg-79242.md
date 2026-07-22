# #79242: File Block: Combine audio/video/image to file transforms

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] File`
- **Merged:** [`a93c84a`](https://github.com/WordPress/gutenberg/commit/a93c84aabe023e8c3a2d4ad141467c167f7d847f)
- **Discussion:** [#79242](https://github.com/WordPress/gutenberg/pull/79242) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The File block now consolidates its block-to-block transforms for `core/audio`, `core/video`, and `core/image` into a single transformation function. This eliminates redundant code while ensuring that captionless audio and video blocks correctly fall back to generating a filename from the media URL, matching the existing behavior of the Image block. The change improves maintainability and standardizes how media blocks convert to File blocks in the block editor.

## Impact

- **Plugin & theme developers:** No action required. The public API for block transforms remains unchanged, and existing `core/audio`, `core/video`, and `core/image` blocks will continue to transform to `core/file` exactly as before.
- **Site owners & editors:** No visible change to the editor UI or transformation behavior.
- **Block authors:** If you maintain custom transforms targeting `core/file`, note that the consolidated function now uses `attributes.src ?? attributes.url` to determine the `href` and `fileName` fallback, which aligns with standard media block attribute conventions.

## Technical details

The change lives in `packages/block-library/src/file/transforms.js`. Previously, the `transforms` object contained three separate `type: 'block'` entries for `core/audio`, `core/video`, and `core/image`, each calling `createBlock( 'core/file', ... )` with nearly identical payloads. The diff merges these into a single entry that accepts all three block names. It introduces `const href = attributes.src ?? attributes.url;` to normalize the source attribute across media and image blocks, then passes `href` to `href`, `fileName` (with the `getFilename()` fallback now applied to all three), and `textLinkHref`. 

Before:
```js
{ type: 'block', blocks: [ 'core/audio' ], transform: ( attributes ) => createBlock( 'core/file', { href: attributes.src, fileName: attributes.caption, ... } ) },
{ type: 'block', blocks: [ 'core/video' ], transform: ( attributes ) => createBlock( 'core/file', { href: attributes.src, fileName: attributes.caption, ... } ) },
{ type: 'block', blocks: [ 'core/image' ], transform: ( attributes ) => createBlock( 'core/file', { href: attributes.url, fileName: attributes.caption || getFilename( attributes.url ), ... } ) },
```

After:
```js
{ type: 'block', blocks: [ 'core/audio', 'core/video', 'core/image' ], transform: ( attributes ) => {
    const href = attributes.src ?? attributes.url;
    return createBlock( 'core/file', { href, fileName: attributes.caption || getFilename( href ), textLinkHref: href, ... } );
} },
```
This reduces the compiled bundle size by 7 bytes and removes ~27 lines of duplicated logic.

## Contribution

Opened and merged by @Mamaduka, the PR required no design debate or alternative approaches; the record carries no detailed discussion beyond the author’s note that AI assisted in drafting the unified transform.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
