# #78829: Boot navigation: wrap items in a list role for valid listitem semantics

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hi0001234d
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Boot`
- **Merged:** [`e380e17`](https://github.com/WordPress/gutenberg/commit/e380e1709fc271aaac7c654330d4693841d7ff38)
- **Discussion:** [#78829](https://github.com/WordPress/gutenberg/pull/78829) · 3 comments · 0 reactions

## Summary

Fixes a structural ARIA violation in the Gutenberg boot navigation component where `listitem` roles lacked a required parent `list` container. This ensures screen readers correctly announce the navigation menu structure and resolves a WCAG 2.1 1.3.1 compliance issue within the Site Editor (`site-editor-v2`). The change introduces only a structural DOM wrapper to align with ARIA 1.2 specifications without altering visual layout or component behavior.

## Impact

- **Site Editors & Accessibility Users**: Navigation menus will now be properly announced as structured lists with accurate item counts by screen readers.
- **Plugin & Theme Developers**: No action required. The modification is isolated to an internal boot package component and does not affect public APIs, block schemas, or REST endpoints.
- **Visual/Layout Impact**: None. The added wrapper is purely semantic and does not interfere with existing CSS flex/grid layouts or keyboard navigation states.

## Technical details

The fix relocates the ARIA `list` wrapper to the parent `Navigation` component in `packages/boot/src/components/navigation/index.tsx`. Previously, items were rendered inside a React fragment (`<>`); they are now wrapped in `<div role="list">` to satisfy the ARIA 1.2 requirement that `listitem` roles must be direct descendants of a `list` (or `group`) parent.

```tsx
// Before
content={
  <>
    { items.map( ( item: MenuItem ) => { ... } ) }
  </>
}

// After
content={
  <div role="list">
    { items.map( ( item: MenuItem ) => { ... } ) }
  </div>
}
```

The wrapper directly encloses the mapped `MenuItem` objects, which render as `NavigationItem`, `DropdownItem`, or `DrilldownItem`. Unit test assertions for `packages/boot/src/components/navigation/navigation-screen/test/index.tsx` are updated to verify the `role="list"` container's presence.

## Contribution

Opened by @hi0001234d to address an ARIA validation bug report (#78803). During code review, @t-hamano advised against wrapping the container at the screen level, recommending instead that the `<div role="list">` be applied directly around `items.map()` in the base `Navigation` component to correctly capture all item variants. The author adopted this structural adjustment before the PR was merged as commit `e380e17`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
