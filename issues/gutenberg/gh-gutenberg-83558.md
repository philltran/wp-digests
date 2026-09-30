# #83558: Media Editor: Fix pan drag flooding undo history after wheel zoom

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`f473ed3`](https://github.com/WordPress/gutenberg/commit/f473ed3033d52437c9f2244126af0bb263c2410e)
- **Discussion:** [#83558](https://github.com/WordPress/gutenberg/pull/83558) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

A pan drag started within the 300 ms debounce window after a wheel zoom in the Media Editor was sharing the still-open wheel gesture. When the debounce timer fired mid-drag, it closed the drag's gesture, causing every remaining pan frame to record its own undo entry. The fix ends the pending wheel gesture explicitly in `handlePointerDown` before the drag begins, so zoom and pan each produce a single undo step.

## Impact

- **Site owners / editors:** No action required. The fix is internal to the Media Editor's interaction controller. Users who zoom with the scroll wheel and then immediately drag to pan will now see undo/redo work in single steps instead of dozens of tiny steps.
- **Plugin & theme developers:** No API change. The `onGestureStart` / `onGestureEnd` callbacks on the interaction controller fire the same number of times per gesture; the fix only prevents a spurious mid-drag `onGestureEnd` that was previously caused by the wheel debounce timer.
- **No configuration, migration, or code changes required.**

## Technical details

The change is in `packages/media-editor/src/image-editor/core/interaction-controller.ts`, inside `InteractionController.handlePointerDown`. Before the fix, if a wheel zoom had started a gesture and its 300 ms debounce timer had not yet elapsed, a subsequent `pointerdown` would begin a drag gesture that shared the same open gesture context. When the timer fired, it called `onGestureEnd`, closing the drag's undo batch; every later `pointermove` then recorded a separate undo entry.

The fix inserts a guard at the top of `handlePointerDown` (after `setPointerCapture`):

```ts
if ( this.wheelGestureActive ) {
    clearTimeout( this.wheelGestureTimer );
    this.wheelGestureActive = false;
    this.options.onGestureEnd?.();
}
```

This clears the pending timer, marks the wheel gesture as inactive, and fires `onGestureEnd` for the zoom *before* the drag's `onGestureStart` is called. The result is two clean undo steps (zoom, then pan) instead of one zoom step plus dozens of per-frame pan steps.

A unit test was added in `packages/media-editor/src/image-editor/core/test/interaction-controller.ts` that uses fake timers to simulate a wheel event, advances 100 ms, fires `pointerdown`, then advances past the 300 ms debounce while continuing to fire `pointermove` events. It asserts that `onGestureEnd` is not called during the drag and that `setPan` is called exactly twice (one per move event).

## Contribution

Opened and merged by @ramonjd with co-authorship from @andrewserong. While testing the fix, @ramonjd identified a related bug in the pixel-snap implementation and filed it as a follow-up PR (#83571). The PR carried only 4 comments and no notable design debate; the fix was a straightforward guard clause.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
