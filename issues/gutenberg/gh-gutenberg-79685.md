# #79685: RTC: Use requestAnimationFrame for awareness cursor resize redraws

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @giteshsarvaiya
- **Labels:** `[Type] Bug`, `[Package] Editor`, `First-time Contributor`, `[Feature] Real-time Collaboration`
- **Merged:** [`e97eb75`](https://github.com/WordPress/gutenberg/commit/e97eb7575553791211825b84f1d1c6774817df4c)
- **Discussion:** [#79685](https://github.com/WordPress/gutenberg/pull/79685) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Replaces the fixed 500ms debounce on resize-triggered awareness cursor and block-highlight redraws with a `requestAnimationFrame`-based recompute. This fixes a Real-Time Collaboration (RTC) UX bug where remote cursors and selection highlights lagged half a second during browser window resizes, snapping into place only after the debounce timer expired instead of tracking layout changes in real time.

## Impact

- **Block editor & RTC users:** Remote collaborator cursors and text selection highlights now update smoothly during window resizes without the previous ~500ms delay.
- **Plugin & theme developers:** No action required. This change is internal to the `@wordpress/editor` package and does not modify public APIs, hooks, or block registration.
- **Hosting & platform teams:** No configuration or migration needed.

## Technical details

- Introduces `useRequestAnimationFrameRecompute` in `packages/editor/src/components/collaborators-overlay/use-debounced-recompute.ts`, which schedules a `requestAnimationFrame` on each call and cancels any pending frame to coalesce rapid resize events.
- Updates `useRenderCursors` and `useBlockHighlighting` to instantiate this new hook alongside the existing `useDebouncedRecompute`, exposing `rerenderCursorsOnResize` and `rerenderHighlightsOnResize` respectively.
- Both hooks now track a separate `resizeToken` state, added to their `useEffect` dependency arrays so resize events trigger immediate recomputation.
- In `packages/editor/src/components/collaborators-overlay/overlay.tsx`, the `onResize` callback now invokes the new RAF-based methods instead of the debounced variants. Mount and periodic redraw paths retain the 500ms debounce.

**Before:**
```tsx
const onResize = useCallback( () => {
    rerenderCursorsAfterDelay();
    rerenderHighlightsAfterDelay();
}, [ rerenderCursorsAfterDelay, rerenderHighlightsAfterDelay ] );
```

**After:**
```tsx
const onResize = useCallback( () => {
    rerenderCursorsOnResize();
    rerenderHighlightsOnResize();
}, [ rerenderCursorsOnResize, rerenderHighlightsOnResize ] );
```

## Contribution

Opened by @giteshsarvaiya and merged after review by @alecgeatches, with co-authorship from @t-hamano. The PR initially explored reducing the debounce value and using a single callback, but the reviewer and author settled on the RAF-based hook for semantic correctness and to avoid arbitrary delay values. Review feedback led to renaming the hook to `useRequestAnimationFrameRecompute`, moving frame cancellation into a `useEffect` cleanup, and renaming internal refs to `requestAnimationFrameRef`. The PR waited for a separate fix to a flaky `tabs.spec.js` e2e test before auto-merging.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
