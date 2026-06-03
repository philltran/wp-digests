# #78636: RTC: Re-render collaborators overlay when the block tree changes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tyxla
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Editor`, `[Package] Block editor`, `[Feature] Real-time Collaboration`, `Collaborative Workflows`
- **Merged:** [`4f30749`](https://github.com/WordPress/gutenberg/commit/4f3074900ad3d0ebb61de9f96f26e7c9c9bb9ddc)
- **Discussion:** [#78636](https://github.com/WordPress/gutenberg/pull/78636) · 8 comments · 0 reactions

## Summary

Fixes a race condition in Real-time Collaboration (RTC) where a joining collaborator's cursor overlay would remain invisible until they moved their cursor. When awareness data arrived before the newly-loaded blocks had been inserted into the DOM, `useRenderCursors` queried `[data-block]` elements, found nothing, and never re-ran because nothing block-related was in its effect dependency array. The fix subscribes to the block tree and adds block client IDs as an effect dependency, forcing a re-render when the block tree changes.

## Impact

- **Plugin & theme developers using RTC/collaborative workflows:** Collaborator cursor overlays now appear immediately on join rather than remaining hidden until the peer moves. No API changes; no action required.
- **Headless & REST consumers:** No impact.
- **Hosting & platform:** No impact.
- No breaking changes, deprecations, or configuration changes.

## Technical details

The affected hook is `useRenderCursors` inside the `@wordpress/block-editor` package. Previously its `useEffect` had no dependency on the block tree, so when Yjs/awareness state arrived before blocks were painted to the DOM, the `[data-block]` querySelectorAll returned an empty set and the effect never re-fired.

The fix adds block client IDs — retrieved from the block editor store — as an explicit dependency of the effect so that it re-runs whenever the block tree is mutated:

```js
// Before: effect deps contained only awareness/cursor state
useEffect(() => {
  renderCursors( /* queries [data-block] */ );
}, [ cursors ]);

// After: block client IDs added as a dep
useEffect(() => {
  renderCursors( /* queries [data-block] */ );
}, [ cursors, blockClientIds ]);
```

Build artifacts changed in `build/scripts/block-editor/index.min.js` (-9 B), `build/scripts/core-data/index.min.js` (-5 B), and `build/scripts/editor/index.min.js` (-114 B). No DB changes, no new hooks or filters, no REST schema changes.

**Note:** Reviewer @jsnajdr flagged that the root cause may be deeper — specifically that `getCursors()` might not be triggering a re-render when its return value changes — and that this fix resolves the symptom by causing the effect to run more frequently rather than fixing the missing reactive subscription. A follow-up investigation was noted as desirable.

## Contribution

Opened and authored by @tyxla. @jsnajdr reviewed and raised a concern that the fix may be masking a deeper missing-dependency bug in `getCursors()` rather than addressing it directly. @tyxla acknowledged this and proposed merging as an unblocking measure for the React 19 upgrade (PR #61521), with a better solution deferred to a follow-up. The PR was merged at commit `4f30749` with @jsnajdr and @Mamaduka as credited co-authors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
