# #79965: Add Expand, Minimize and Hide Notes modes to the Options (ellipsis) menu

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Notes`
- **Merged:** [`5bcae66`](https://github.com/WordPress/gutenberg/commit/5bcae66af6179cd6a5b35994876c3f3ba53f521a)
- **Discussion:** [#79965](https://github.com/WordPress/gutenberg/pull/79965) · 18 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg editor now supports three display modes for floating collaboration notes: full threads, minimized avatar pills, and hidden. Users can switch between these modes via a new "Notes" group in the editor's Options (ellipsis) menu, and the canvas automatically collapses threads to pills or hides them entirely as the editor width narrows. This gives editors finer control over canvas real estate without losing access to threaded feedback.

## Impact

- **Editor users & content creators:** Gain explicit control over floating notes visibility and canvas width. No action required; behavior is opt-in via the menu or automatic based on viewport.
- **Plugin & theme developers:** No breaking changes or removed APIs. If you extend or override `FloatingNotes` or `NoteThread`, you should handle the new `isCompact` prop to maintain correct compact rendering.
- **Hosting & platform teams:** No action required. The change is contained within the editor package and does not affect server-side rendering, REST API, or database schemas.

## Technical details

The change introduces `notesDisplayMode` state (`'full'`, `'minimized'`, `'hidden'`) in `packages/editor/src/components/collab-sidebar/index.js`, wired to radio options in the Options menu. `FloatingNotes` in `floating-notes.js` now accepts an `isCompact` prop and calculates `reservedWidth` dynamically, applying `padding-inline-end` to the canvas root via inline CSS. Thresholds in `constants.js` (`MIN_CANVAS_WIDTH_FOR_FULL_NOTES`, `MIN_CANVAS_WIDTH_FOR_FLOATING_NOTES`) drive automatic responsive collapse. `NoteThread` in `note-thread.js` conditionally renders a `NoteCard` instead of the full thread UI when `isCompact` is true and the thread isn't selected. CSS in `style.scss` adds `.is-compact` rules to anchor pills to the inline edge and hide metadata until `:hover` or `:focus-within`. Keyboard shortcuts (`access+J`, `access+I`, `access+E`, `access+A`) are registered per mode to toggle the display state.

## Contribution

Opened and merged by @adamsilverstein with co-authors @jasmussen, @annezazu, @manzoorwanijk, @t-hamano, and @jeffpaul. The approach evolved after earlier attempts were paused pending design alignment on a resizable sidebar; this iteration was rebuilt on the canvas-margin architecture from #79864 and moved the controls into the Options menu per reviewer feedback. Keyboard shortcuts were adapted from Docs' two-key chords to WordPress conventions to avoid conflicts, and the menu order was reversed to follow a none-to-all progression.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
