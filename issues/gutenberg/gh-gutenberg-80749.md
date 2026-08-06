# #80749: Block Editor: Fix floated blocks overlapping sticky blocks

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] Base styles`
- **Merged:** [`eb992ec`](https://github.com/WordPress/gutenberg/commit/eb992ec35415c6a39174b8d71403d09ea81c7757)
- **Discussion:** [#80749](https://github.com/WordPress/gutenberg/pull/80749) · 1 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Block Editor’s base styles now render floated (left/right aligned) blocks below sticky and fixed-position blocks, aligning the editing canvas with front-end rendering. Previously, floated blocks used a `z-index` of `21`, causing them to visually overlap sticky blocks while scrolling the editor. This fix lowers the editor z-index to `9` so the editing experience accurately reflects the published site.

## Impact

- **Block editor users & content authors**: Floated blocks will no longer visually obscure sticky groups or containers while scrolling the canvas.
- **Theme & plugin developers**: No code changes required. If you override base z-index values for floated blocks in your own stylesheets, verify they still sit above adjacent content and resize handles.
- **Platform/Hosting**: No action required.

## Technical details

The change modifies `packages/base-styles/_z-index.scss`. The selector `{core/image aligned left or right} .wp-block` had its z-index value reduced from `21` to `9`. Sticky and fixed blocks are assigned `z-index: 10` via the position block support, so the new value ensures floated blocks paint below them while remaining above `z-index: auto` adjacent blocks and in-block UI like resize handles (`z-index: 2`). 

Before:
```scss
"{core/image aligned left or right} .wp-block": 21,
```

After:
```scss
"{core/image aligned left or right} .wp-block": 9,
```

## Contribution

Opened to resolve issue #62774, the PR moved through review with co-authors @Mamaduka, @talldanwp, and @rohitmathur7. The author noted that AI assistance was used to draft the fix and description, but manual review verified the z-index hierarchy preserved block selection functionality. The change was merged directly after a single review cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
