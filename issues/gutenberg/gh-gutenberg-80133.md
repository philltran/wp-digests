# #80133: useMergeRefs: apply ref changes after out-of-render attachment

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Compose`
- **Merged:** [`db2ff81`](https://github.com/WordPress/gutenberg/commit/db2ff815a4b14ee828d53e6f12e711c786b9ff1f)
- **Discussion:** [#80133](https://github.com/WordPress/gutenberg/pull/80133) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the `useMergeRefs` hook where ref updates were skipped if the target element attached outside the calling component's render cycle. This previously left stale callbacks on the element when a child component mounted it in a separate commit.

## Impact

- **Plugin & theme developers** using `@wordpress/compose`'s `useMergeRefs`: **No action required**. The hook now correctly handles refs passed to children that mount elements asynchronously or in their own commit cycles.
- **Gutenberg/Block Editor developers**: Resolves a scenario where merged refs (e.g., typing observers) could miss listener swaps when the canvas body is portaled or mounted by a child component.

## Technical details

In `packages/compose/src/hooks/use-merge-refs/index.ts`, the implementation replaces the `isAttachedRef`, `didElementChangeRef`, and `previousRefsRef` state variables with a single `attachedRefsRef`. The `useLayoutEffect` now diffs the incoming `refs` array directly against `attachedRefsRef.current` to detach stale callbacks and attach new ones. The ref callback itself updates `attachedRefsRef.current` on mount and unmount. This removes one `useLayoutEffect` and two `useRef` calls, simplifying the hook's dependency tracking. A regression test verifies that dependency changes correctly swap refs even when the element mounts in a child-only commit.

## Contribution

Opened and merged by @ellatrix. The issue was discovered while debugging a typing observer swap failure in the BlockCanvas (#80131). The PR simplifies the hook's internal state tracking to avoid race conditions when elements attach outside the parent's render cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
