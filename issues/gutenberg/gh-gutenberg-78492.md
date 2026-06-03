# #78492: Compose: Simplify subscribeDelegatedListener root detection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Performance`, `[Package] Compose`
- **Merged:** [`1a09e39`](https://github.com/WordPress/gutenberg/commit/1a09e39a835841b6c8de0c239dd17885ee05d3bb)
- **Discussion:** [#78492](https://github.com/WordPress/gutenberg/pull/78492) · 3 comments · 0 reactions

## Summary

The `subscribeDelegatedListener` utility in the `@wordpress/compose` package now uses a consolidated root detection expression instead of a multi-branch helper function. By leveraging a single `(target as Node).ownerDocument ?? target` lookup, the code eliminates verbose type-checking while preserving identical event dispatch behavior across Elements, Documents, and Window objects.

## Impact

- **Plugin & theme developers**: No action required. The internal root-detection logic changed, but the public API contract for `subscribeDelegatedListener` remains unchanged with zero behavioral differences.
- **Hosting & platform teams**: No migration or configuration changes needed. This is a performance-oriented code simplification that does not affect server-side environments, PHP hooks, or REST routes.

## Technical details

The change modifies `packages/compose/src/utils/subscribe-delegated-listener/index.ts`. The previous `getRoot(target)` helper relied on explicit duck typing (`nodeType === 9` for Documents) and cross-realm safe checks for `Window`, followed by an Element fallback. This was replaced with a single coalescing expression:

```ts
// Before (simplified from diff)
function getRoot( target: EventTarget ): EventTarget {
	if ( target.nodeType === 9 ) return target;
	if ( window.window === target ) return target;
	return (target as Node).ownerDocument ?? target;
}

// After
const root = (target as Node).ownerDocument ?? target;
```

The new approach encodes all three target types in one lookup: `Element` nodes resolve to their `ownerDocument`; `Document` nodes resolve to themselves because `document.ownerDocument` is strictly `null` (triggering the coalesce to `target`); and `Window` objects resolve to themselves because `window.ownerDocument` is `undefined` (also triggering the coalesce). This eliminates cross-realm `instanceof` hazards and removes a redundant `isWindow` derivation. All 12 existing unit tests covering cross-realm iframes and event dispatching pass without modification.

- **Breaking changes**: None.
- **Performance impact**: Minor execution optimization from reduced branching (~23 B minified size reduction reported).

## Contribution

Authored and merged by @ellatrix as a follow-up to `#78310`. The pull request focused on code hygiene within the `@wordpress/compose` package, specifically targeting the `subscribeDelegatedListener` utility. Review confirmed that the standardized `Node.ownerDocument` behavior across DOM levels safely replaces the previous branching logic without introducing new edge cases or cross-realm failures.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
