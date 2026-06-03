# #78685: Compose: Support React 19 ref callback cleanups in `useMergeRefs`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tyxla
- **Labels:** `[Type] Enhancement`, `[Package] Compose`
- **Merged:** [`a19d83d`](https://github.com/WordPress/gutenberg/commit/a19d83d75c698587cc62c77ee8f2ce49ee29109a)
- **Discussion:** [#78685](https://github.com/WordPress/gutenberg/pull/78685) · 2 comments · 1 reactions

## Summary

`useMergeRefs` in `@wordpress/compose` now supports the React 19 ref callback cleanup contract. When an inner callback ref returns a function, `useMergeRefs` stores that cleanup and invokes it at teardown (unmount, node change, or dependency change) instead of re-invoking the callback with `null`. Callbacks that do not return a cleanup retain the existing `null`-pass behavior unchanged, so this is a purely additive change. The update is a prerequisite for a planned deprecation of `useRefEffect` in favor of native React 19 `useCallback` ref cleanups, tracked in [#71336](https://github.com/WordPress/gutenberg/issues/71336).

## Impact

**Plugin & theme developers**
- No action required. All 132 existing `useMergeRefs` call sites are unaffected; void-returning callbacks continue to receive `null` at teardown as before.
- Ref callbacks passed to `useMergeRefs` that return a cleanup function will now have that cleanup correctly invoked, enabling the React 19-native `useCallback` ref pattern as a first-class alternative to `useRefEffect`.

**`useRefEffect` users**
- No change in this PR. `useRefEffect` source, tests, and call sites are explicitly out of scope and untouched. A future PR will handle the deprecation.

## Technical details

Changes are in `packages/compose/src/hooks/use-merge-refs/index.ts`.

**`assignRef`** is updated to return the ref callback's return value, now typed as `(() => void) | undefined`. Object refs (`.current` assignment) continue to return `undefined`.

A new **`cleanupsRef`** (`useRef<Array<(() => void) | undefined>>`) is introduced, position-indexed to match the inner refs array. When `assignRef` is called with a function ref that returns a cleanup, that cleanup is stored at `cleanupsRef.current[i]`.

Two **teardown paths** are updated:
1. The outer-callback `null` path (detach from DOM node).
2. The `useLayoutEffect` cleanup path (dependency change).

In both paths, if `cleanupsRef.current[i]` is populated, the cleanup is called and the slot cleared — the inner ref is **never** called with `null`. If no cleanup exists, the previous `ref(null)` behavior is preserved. This matches React 19's own invariant for native ref callbacks.

**JSDoc** example updated from `useRefEffect` to the React 19 `useCallback` pattern:

```js
// Before (useRefEffect pattern)
const ref = useRefEffect( ( node ) => {
    node.addEventListener( 'click', handler );
    return () => node.removeEventListener( 'click', handler );
}, [ handler ] );

// After (React 19 native cleanup — now honored by useMergeRefs)
const ref = useCallback( ( node ) => {
    if ( ! node ) return;
    node.addEventListener( 'click', handler );
    return () => node.removeEventListener( 'click', handler );
}, [ handler ] );

const mergedRef = useMergeRefs( [ ref, forwardedRef ] );
```

Test suite expanded from 7 to 14 tests; the 7 new tests cover cleanup invocation on unmount, node change, dependency change, simultaneous node+dep change, ref-becoming-disabled, mixed cleanup+void refs, and mixed cleanup+object refs. Bundle delta: `build/scripts/compose/index.min.js` +55 B (+0.47%).

## Contribution

Opened and authored by @tyxla as one checkbox in the broader `useRefEffect` deprecation effort (#71336). @jsnajdr, @Mamaduka, and @ciampo are credited as co-contributors via the merge commit. The PR generated only 2 comments and 1 reaction, with no recorded design debate over alternative approaches. The scope was deliberately kept narrow — `useRefEffect` migration, existing call sites, and the native build re-export were all explicitly deferred to follow-on PRs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
