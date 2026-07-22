# #80553: Device type preview: fix collapsing to content height

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`
- **Merged:** [`749b2f0`](https://github.com/WordPress/gutenberg/commit/749b2f0a00608496881c9826406cb6e7adee323e)
- **Discussion:** [#80553](https://github.com/WordPress/gutenberg/pull/80553) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a regression in the block editor where device type previews (tablet/mobile) collapse to the content height instead of maintaining the correct device viewport height when the canvas is empty or contains minimal content. The fix ensures the iframe body respects the configured canvas height, restoring the expected responsive preview behavior from WordPress 7.0.

## Impact

- **Block theme & plugin developers:** No direct API changes, but visual testing of responsive layouts in the editor will now behave correctly on empty or sparse posts/templates.
- **Site owners & editors:** Device preview dropdowns will display the correct viewport dimensions regardless of post content length.
- **No action required.** This is a visual/editor UX fix with no code changes needed.

## Technical details

The diff lifts `canvasWidth` and `canvasHeight` from `packages/editor/src/components/resizable-editor/index.js` up to `packages/editor/src/components/visual-editor/index.js`, where they are now passed as explicit `width` and `height` props. `ResizableEditor` was refactored to accept `onResizeStart` and `onResizeStop` callbacks to track manual drag state via a local `isResizingCanvas` state.

The caller pattern changed from:
```jsx
<ResizableEditor enableResizing={ enableResizing } height="100%">
```
to:
```jsx
<ResizableEditor
  enableResizing={ enableResizing }
  width={ enableResizing && canvasWidth ? canvasWidth + 'px' : '100%' }
  height={ enableResizing && canvasHeight && ! isResizingCanvas ? canvasHeight + 'px' : '100%' }
  onResizeStart={ () => setIsResizingCanvas( true ) }
  onResizeStop={ () => setIsResizingCanvas( false ) }
>
```

In `VisualEditor`, a new CSS rule conditionally applies `min-height:100vh;` to `.block-editor-iframe__body` when `canvasHeight` is defined, the post type is not resizable (navigation, template part, or pattern), and the user isn't actively dragging. This prevents the iframe body from collapsing while preserving the intentional content-height collapse during manual resizing introduced in #80271.

## Contribution

Opened by @talldan as a targeted follow-up to #80271. During review, @ramonjd noted that the canvas height still collapses during manual drag operations, which the author confirmed was an intentional carryover from the parent PR to maintain existing behavior, with potential aspect-ratio or full-height refinements slated for a future follow-up. The change was validated by @noruzzamans and @andrewserong before merging and was subsequently backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
