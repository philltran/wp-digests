# #74596: iOS: remove jumping hack, add typewriter

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Edit Post`, `Backported to WP Core`
- **Merged:** [`795dcfb`](https://github.com/WordPress/gutenberg/commit/795dcfb377f8911918ceea8eadc01a702d5b5ad6)
- **Discussion:** [#74596](https://github.com/WordPress/gutenberg/pull/74596) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor removes a long-standing iOS Webkit scroll workaround that previously forced the viewport back to the top on every scroll event. This hack was originally introduced to keep the mobile toolbar visible, but it caused erratic jumping behavior that disrupted the editing experience. Removing it restores natural scrolling on iOS devices, at the cost of the toolbar occasionally scrolling out of view when pressing Enter.

## Impact

- **Plugin & theme developers / block authors:** No direct code changes required. The removed logic was internal to the `@wordpress/edit-post` package.
- **iOS mobile editors:** Users will experience smoother, native scrolling without the previous forced viewport resets. The top toolbar may scroll out of view during editing, requiring manual scrolling to access.
- **Hosting & platform teams:** No action required. This is a client-side behavior change in the editor package.

## Technical details

The diff removes a 30-line block from `packages/edit-post/src/index.js` inside the `initializeEditor` function. The removed code attached a `scroll` event listener to `window` that checked for `window.navigator.userAgent.indexOf( 'iPhone' ) !== -1`. When triggered on the `document` target with `window.scrollY > 100`, it manually adjusted `editorScrollContainer.scrollTop` (targeting `.interface-interface-skeleton__body`) and called `window.scrollTo( 0, 0 )` in visual mode. The removal eliminates this forced scroll synchronization, allowing the browser's native scroll behavior to take over. No new hooks, filters, or public APIs are introduced or deprecated.

## Contribution

Opened by @ellatrix and merged with co-authorship from @jasmussen, the change addresses accumulated complaints about the iOS toolbar workaround causing disruptive jumping (tracked in #63844). After reviewing video evidence of the trade-offs, @jasmussen endorsed the removal, noting that Safari updates may have mitigated the original off-screen toolbar issue. The PR was backported to the `wp/7.1` branch for inclusion in the upcoming release cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
