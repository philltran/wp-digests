# #82598: Rich text: restore contenteditable on pointercancel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tyxla
- **Labels:** `[Type] Bug`, `[Package] Rich text`
- **Merged:** [`10fc677`](https://github.com/WordPress/gutenberg/commit/10fc677c970ae94b5ac423df5c8e077d9f9ae2db)
- **Discussion:** [#82598](https://github.com/WordPress/gutenberg/pull/82598) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/rich-text` package's `preventFocusCapture` listener now also handles `pointercancel`, not just `pointerup`. Previously, a touch that turned into a scroll fired `pointercancel`, so the `contenteditable` attribute that had been set to `false` on `pointerdown` was never restored, leaving text blocks uneditable until the editor re-rendered. The fix subscribes the existing restore handler to `pointercancel` as well.

## Impact

- **Site owners / editors on touch devices:** Swiping to scroll (per the PR's testing steps, along the edge of the display outside the block canvas) no longer leaves rich text blocks stuck in a non-editable state.
- **Plugin & theme developers:** No API changes and no action required. Any custom block using the `RichText` component / `useRichText` hook gets the fix automatically once it picks up the updated package.
- **Mobile / embedded editor consumers:** GutenbergKit carried a downstream patch over the published package for this ([wordpress-mobile/GutenbergKit#135](https://github.com/wordpress-mobile/GutenbergKit/pull/135)); the PR states this ports it upstream so that patch can be dropped.

## Technical details

The change is in `packages/rich-text/src/hook/event-listeners/prevent-focus-capture.js`. `preventFocusCapture` sets `contenteditable` to `false` on `pointerdown` (to avoid focus capture when clicking around a flex parent) and restores it in `onPointerUp` on `pointerup`.

The diff adds a third delegated subscription using the same handler, and tears it down with the others in the cleanup function:

```js
const unsubscribePointerCancel = subscribeDelegatedListener(
	defaultView,
	'pointercancel',
	onPointerUp
);
return () => {
	unsubscribePointerDown();
	unsubscribePointerUp();
	unsubscribePointerCancel();
};
```

A `### Bug Fixes` entry was also added to `packages/rich-text/CHANGELOG.md`. The diff includes no new tests. The bundle-size bot reports `build/scripts/rich-text/index.min.js` at -2 B.

## Contribution

Authored by @tyxla, who noted the fix originated in GutenbergKit and cites #72230 and #67986 as issues it attempts to address. @ellatrix approved it as safe regardless of anything else, and @tyxla thanked @dcalhoun for pointing to the prior downstream fix. The PR description notes it was generated with Claude Code.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
