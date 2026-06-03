# #69087: Reset zoom level on component unmount

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Infinite-Null
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Zoom Out`
- **Merged:** [`63bcc8b`](https://github.com/WordPress/gutenberg/commit/63bcc8ba41ba630fb8fe8ed14e75c059eca33ef1)
- **Discussion:** [#69087](https://github.com/WordPress/gutenberg/pull/69087) · 14 comments · 0 reactions

## Summary

The `useScaleCanvas` hook in the block editor now prevents stale zoom states during viewport resizes by guarding against missing `iframeDocument` references. This synchronizes the zoom toggle button state with the actual canvas scale when the editor component remounts across responsive breakpoints.

## Impact

- **Site editors & plugin developers:** Eliminates visual mismatch where the zoom-out toggle remained in a pressed state after resizing the browser window to a smaller width and back.
- **Block editor consumers:** No code changes or configuration required. The fix is isolated to the internal hook and transparent to themes, plugins, or headless clients relying on standard editor components.

## Technical details

- Modified `packages/block-editor/src/components/iframe/use-scale-canvas.js`.
- Replaced a short-circuit evaluation inside the zoom transition trigger with an explicit early return guard:
  ```js
  // Before:
  const trigger = iframeDocument && previousIsZoomedOut.current !== isZoomedOut;
  
  // After:
  if ( ! iframeDocument ) {
      return;
  }
  const trigger = previousIsZoomedOut.current !== isZoomedOut;
  ```
- This ensures that when the canvas component remounts during a viewport resize, any pending zoom state transitions are properly evaluated once the `iframeDocument` becomes available. Previously, rapid resizing could cause `iframeDocument` to be null at different render timesteps than state updates, desyncing the UI toggle from the applied CSS scale.

## Contribution

Opened by @Infinite-Null as PR #69087. Initial implementation attempted a cleanup effect on component unmount, but reviewer feedback from @Mamaduka and @t-hamano traced the actual root cause to the `useScaleCanvas` hook's timing relative to `iframeDocument` availability during viewport changes. The discussed unmount cleanup approach was replaced with the final guarded `useEffect` pattern shipped in commit `63bcc8b`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
