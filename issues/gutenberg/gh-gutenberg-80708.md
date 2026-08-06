# #80708: Writing flow: stop the page scrolling on caret moves within blocks taller than the viewport

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Feature] Writing Flow`, `[Package] DOM`, `Backported to WP Core`
- **Merged:** [`a59a875`](https://github.com/WordPress/gutenberg/commit/a59a87575f2589e2a62ef12778fcb4079bfa8247)
- **Discussion:** [#80708](https://github.com/WordPress/gutenberg/pull/80708) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a writing flow bug where pressing arrow keys inside a block taller than the viewport (e.g., a long Code or Quote block) caused the editor page to scroll on every keystroke. The issue originated from the `isEdge` function repeatedly scrolling the block into view to measure whether the caret was at the container's boundary. The fix adds a viewport boundary check that short-circuits the measurement when the edge is off-screen but the selection remains visible, eliminating the lurching scroll behavior.

## Impact

- **Block & theme developers:** No action required. This is an internal DOM/writing flow fix with no public API changes.
- **Site owners & editors:** No action required. The editor experience improves automatically on update.
- **Plugin developers:** No action required. The change is confined to Gutenberg's internal `@wordpress/dom` package and does not expose or modify public hooks, filters, or block APIs.
- **Breaking changes / migration:** None. The behavior change is strictly internal to the block editor's caret navigation logic.

## Technical details

The diff modifies `packages/dom/src/dom/is-edge.js`. Previously, `isEdge` unconditionally called `scrollIfNoRange` with a callback to `hiddenCaretRangeFromPoint` to determine if the selection was at the container's edge. For blocks exceeding the viewport height, this triggered a scroll on every arrow press. The updated logic introduces two boolean checks (`isFarFromVerticalEdge` and `isFarFromHorizontalEdge`) that compare the calculated edge coordinates (`x`, `y`) against `defaultView.innerHeight` and `defaultView.innerWidth`. If the edge coordinate falls outside the viewport while the selection rectangle (`rangeRect`) remains fully within it, the function immediately returns `false`, bypassing the scroll-and-measure fallback. An e2e test in `test/e2e/specs/editor/various/writing-flow.spec.js` verifies that no `scroll` events fire during arrow key navigation in a 70-line Code block.

## Contribution

The PR addresses two linked issues (#73801 and #68492) by introducing a geometric short-circuit rather than refactoring the entire writing flow measurement pipeline. The author documented a narrow trade-off: skipping the probe means a few pixels of text just outside the viewport may not trigger edge detection until the page naturally scrolls, but this is preferable to the previous full-viewport jump on every press. The change was reviewed, validated with cross-engine e2e tests, and backported to the `wp/7.1` branch for inclusion in the upcoming WordPress release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
