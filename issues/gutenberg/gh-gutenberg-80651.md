# #80651: Rich text: remove tabIndex from editable elements again to fix shift+click selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`5d2a11f`](https://github.com/WordPress/gutenberg/commit/5d2a11f3bc7cce1afa7c1dae17422180e641cb25)
- **Discussion:** [#80651](https://github.com/WordPress/gutenberg/pull/80651) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This pull request restores the removal of `tabIndex` from all rich text editable elements in the block editor, fixing a regression that broke cross-block `Shift`+click text selection in every browser. The change also prevents the editor viewport from scrolling unexpectedly when extending a selection across a long document. By ensuring the editing host remains focused without triggering native browser focus behavior, the block editor can rely on the browser’s native selection extension logic.

## Impact

- **Block editor users & developers:** No action required. The regression in `Shift`+click selection across blocks (introduced in #79105) is resolved, and viewport scrolling during cross-block selection is fixed.
- **Plugin & theme developers:** No breaking changes or public API modifications. Internal block editor components were adjusted, but no external-facing hooks, filters, or block.json fields were altered.
- **Hosting & platform teams:** No configuration or migration steps needed. The fix is backported to WordPress 7.1.

## Technical details

The diff modifies three files in `packages/block-editor/src/components/`:
- `rich-text/index.js`: Changes the `tabIndex` removal condition from `hasEditableRoot && props.tabIndex === 0` to simply `props.tabIndex === 0`, ensuring non-`editableRoot` blocks (like headings or verse) no longer render with `tabindex="0"`.
- `writing-flow/use-click-selection.js`: Introduces `getSelectionStart()` to detect whether the anchor block contains an active text selection. If not (e.g., an image or spacer block), the handler manually sets a native `Selection` anchor at the near edge of the selected block before the click event fires, preventing browsers from synthesizing an incorrect anchor.
- `writing-flow/utils.js`: Updates `setContentEditableWrapper()` to call `node.focus( { preventScroll: true } )`, stopping the viewport from jumping to the top of the canvas when focusing the editing host during cross-block selection.

Before/after pattern for the focus call:
```js
// Before
node.focus();
// After
node.focus( { preventScroll: true } );
```
The changes rely on native DOM selection APIs (`document.getSelection()`, `selection.setPosition()`) and do not introduce new public hooks or filters.

## Contribution

Opened to address a regression introduced in #79105 that accidentally limited `tabIndex` removal to `editableRoot` blocks, breaking cross-block selection in Safari and other browsers. The author introduced a manual anchor-positioning strategy in `use-click-selection.js` to handle cases where selection starts from non-text blocks, after evaluating that native browser behavior would otherwise synthesize incorrect anchors. The PR was reviewed by @noruzzamans and @jasmussen, with testing confirmed across Chromium, Firefox, and real Safari via safaridriver. The change was merged and subsequently cherry-picked to the `wp/7.1` branch for inclusion in the upcoming release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
