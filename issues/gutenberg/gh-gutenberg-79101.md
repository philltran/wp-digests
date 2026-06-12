# #79101: Media Editor Modal: Add a loading and simple error state

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`d2580c7`](https://github.com/WordPress/gutenberg/commit/d2580c744241c77a4a50ae281502df69471b5e74)
- **Discussion:** [#79101](https://github.com/WordPress/gutenberg/pull/79101) · 5 comments · 1 reactions

## Summary

This update improves the Media Editor Modal's UX on slow or flaky connections by introducing explicit loading and error states for image cropping. Instead of displaying drag handles before the underlying image fully streams in, the modal now renders a spinner, fades the cropper in once pixels are ready, and gracefully handles failed loads with an alert message.

## Impact

- **Editor users & site owners**: Experience smoother media editing with no jarring layout shifts or floating crop handles appearing prematurely.
- **Plugin & theme developers**: No breaking changes or API modifications. The update is strictly internal to the `@wordpress/media-editor` package and affects only the frontend rendering of the block editor's crop tool.
- **Action required**: None.

## Technical details

- In `packages/media-editor/src/components/media-editor-canvas/index.tsx`, a new `status` state (`'loading' | 'loaded' | 'error'`) tracks actual pixel readiness.
- A `useEffect` hook deploys a `window.Image()` probe to fire `onload`/`onerror` callbacks. It immediately checks `probe.complete` to handle cached images, and cleans up listeners on unmount.
- The `<Cropper>` is wrapped in a container that conditionally applies the `.is-loaded` class (via `clsx`), gating both visual transparency (`opacity: 0` → `1`) and interactivity (`pointer-events`).
- Focus behavior was updated to `focusOnMount={ focusOnMount && status === 'loaded' }` so the cropper's focus effect only triggers after the image reveals.
- An early return renders a `<Spinner />` overlay during `'loading'`, while an error fallback returns a `<div className="media-editor-canvas__error" role="alert">` with a translated paragraph.
- `style.scss` imports base animation utilities, positions the spinner absolutely, and implements a 0.2s fade-in animation that respects `prefers-reduced-motion`.

## Contribution

Opened by @andrewserong to resolve clunky UI behavior when crop tools open on slow connections. Review feedback from @ramonjd prompted adjustments to focus gating and pointer-event handling before merge. The commit finalizes a lightweight loading/blur pattern rather than a broader error-handling refactor, which was noted as scope for follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
