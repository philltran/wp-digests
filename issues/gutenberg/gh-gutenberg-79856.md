# #79856: Tabs: fix stale overflow fade as content settles on load

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @enejb
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`ad8c485`](https://github.com/WordPress/gutenberg/commit/ad8c48522d0f6b0a4629e0fe81a47766285e822c)
- **Discussion:** [#79856](https://github.com/WordPress/gutenberg/pull/79856) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `Tabs` component in `@wordpress/ui` no longer leaves its horizontal-overflow fade stuck over tabs when content reflows after initial mount. Previously, overflow detection only observed the tab list container, so sub-pixel layout shifts from web fonts or async label resolution would freeze the fade state. The fix observes each individual tab alongside the list, ensuring the scroll affordance updates in real time as content settles.

## Impact

- **Plugin & theme developers / UI builders:** No code changes required. The `Tabs.List` and `Tabs.Tab` APIs remain identical, but the component now correctly hides the edge fade when tabs fit after initial load.
- **Agency & platform teams:** No migration or configuration needed. This is a client-side rendering fix with no server-side or REST implications.
- **No breaking changes or deprecations.**

## Technical details

The change modifies `packages/ui/src/tabs/list.tsx` to replace a list-only `ResizeObserver` with a dual-observer strategy. The `observeTabs` function now disconnects and re-observes both `listEl` and all `[role="tab"]` descendants. A `MutationObserver` with `{ childList: true, subtree: true }` tracks dynamic tab insertion/removal to keep the observers in sync. The `setOverflow` call was converted to a functional updater to prevent stale closure issues during rapid reflows:

```tsx
// Before
setOverflow( { first: ..., last: ..., isScrolling: ... } );

// After
setOverflow( ( prev ) =>
  prev.first === next.first &&
  prev.last === next.last &&
  prev.isScrolling === next.isScrolling
    ? prev
    : next
);
```
A new regression test in `packages/ui/src/tabs/test/overflow.test.tsx` mocks `ResizeObserver` and `scrollWidth` to verify that firing only tab-level observers clears the `is-overflowing-last` CSS class when content fits.

## Contribution

Opened by @enejb and merged following review from @simison, @ciampo, and @aduth. The discussion evaluated three approaches: a frame-based settle loop (rejected as brittle), an `IntersectionObserver` on edge tabs (noted as previously used but incompatible with the Base UI migration), and the final solution of mirroring Base UI by observing each tab box. The tab-observation approach was adopted for its event-driven reliability across browsers.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
