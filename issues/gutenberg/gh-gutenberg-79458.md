# #79458: UI: Use isomorphic layout effects for SSR

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`1d5c9df`](https://github.com/WordPress/gutenberg/commit/1d5c9df35bb9861f5594385b87de740dd6b0023c)
- **Discussion:** [#79458](https://github.com/WordPress/gutenberg/pull/79458) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Replaces direct `useLayoutEffect` calls in `@wordpress/ui` with `useIsomorphicLayoutEffect` from `@wordpress/compose` to eliminate React SSR warnings. The change ensures layout effects run synchronously in the browser but fall back to `useEffect` during server rendering, preventing console noise without altering runtime behavior.

## Impact

- **Plugin & theme developers**: Fewer React warnings in SSR environments when consuming `@wordpress/ui` hooks.
- **Storybook/development workflows**: Resolves layout effect warnings in the Popover story utilities.
- **No action required**: This is a non-breaking internal refactor that preserves existing behavior and requires no migration.

## Technical details

The diff swaps `useLayoutEffect` for `useIsomorphicLayoutEffect` in two locations within `@wordpress/ui`. In `packages/ui/src/utils/use-overlay-scroll-state-attributes.ts`, the `useOverlayScrollStateAttributes` hook now imports `useIsomorphicLayoutEffect` from `@wordpress/compose` instead of `@wordpress/element`. Similarly, `packages/ui/src/popover/stories/utils.tsx` updates the `useMeasure` utility to use the isomorphic variant. The change is a direct 1:1 replacement in the callback signatures, preserving the original measurement and scroll-state logic while deferring execution to `useEffect` when `window` is undefined.

**Before:**
```tsx
import { useLayoutEffect } from '@wordpress/element';
// ...
useLayoutEffect( () => { ... } );
```

**After:**
```tsx
import { useIsomorphicLayoutEffect } from '@wordpress/compose';
// ...
useIsomorphicLayoutEffect( () => { ... } );
```

## Contribution

Opened by @ciampo as a follow-up to #78678 to address remaining raw layout effects in the UI package. Reviewed and merged by @mirka with zero compressed-size impact. The PR moved quickly through review with no alternative approaches debated, as the isomorphic hook was already established in `@wordpress/compose`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
