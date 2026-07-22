# #79255: Grid overlays: Use canvas iframe window for viewport visibility detection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Layout`
- **Merged:** [`714482b`](https://github.com/WordPress/gutenberg/commit/714482b47208aea137166cdfbbfe417249d215ee)
- **Discussion:** [#79255](https://github.com/WordPress/gutenberg/pull/79255) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Grid block overlay now correctly hides grid lines and resize handles when a block is hidden at the current browser viewport width, rather than relying solely on the selected device preview mode. Previously, manually resizing the browser window would leave the editing UI visible even when the underlying block was hidden. This fix aligns the overlay's visibility logic with the actual canvas iframe viewport, matching the behavior already used by the block visibility badge.

## Impact

- **Block editor users & developers:** No code changes required. The Grid block's visual editing UI now accurately reflects visibility settings during manual browser resizing.
- **Plugin/theme developers using `useBlockVisibility`:** The hook now accepts an optional `view` parameter to tie visibility detection to a specific iframe window. Existing calls without `view` continue to work but may not reflect manual resize events accurately.
- **No action required** for site owners or standard theme/plugin setups.

## Technical details

The diff modifies `packages/block-editor/src/hooks/grid-visualizer.js` and `packages/block-editor/src/hooks/layout-child.js`. Both files previously derived visibility state from `deviceType` alone. The change extracts the block's DOM element via `useBlockElement( clientId )`, derives the canvas iframe window (`blockElement?.ownerDocument?.defaultView`), and passes it as `view: canvasView` to `useBlockVisibility`. The hook now returns `currentViewport`, which is used to call `isBlockParentHiddenAtViewport( clientId, currentViewport )` for accurate ancestor visibility checks. This replaces the previous `deviceType`-only ancestor check that failed to update on manual resize.

Before/after pattern in `grid-visualizer.js`:
```javascript
// Before
const { isBlockCurrentlyHidden } = useBlockVisibility( {
    blockVisibility,
    deviceType,
} );

// After
const blockElement = useBlockElement( clientId );
const rawCanvasView = blockElement?.ownerDocument?.defaultView;
const canvasView = rawCanvasView === null ? undefined : rawCanvasView;
const { isBlockCurrentlyHidden, currentViewport } = useBlockVisibility( {
    blockVisibility,
    deviceType,
    view: canvasView,
} );
```
The `useBlockVisibility` hook's `view` argument now drives viewport detection against the correct iframe context, ensuring the overlay respects actual browser resize events.

## Contribution

Opened by @t-hamano as a follow-up to earlier visibility refactor PRs. During review, @tellthemachines identified that sidebar state was interfering with visibility detection and that ancestor blocks hidden at a viewport weren't properly hiding the visualizer. @t-hamano addressed both in subsequent commits by passing the canvas view to `useBlockVisibility` and introducing a `currentViewport`-driven ancestor check. The PR was co-authored by @ramonjd and @tellthemachines during review and merged without further debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
