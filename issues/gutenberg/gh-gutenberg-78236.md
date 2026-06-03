# #78236: Add custom widget dashboard resize handle styling.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`, `[Package] Grid`
- **Merged:** [`71498dd`](https://github.com/WordPress/gutenberg/commit/71498dd6435e90a092844808fad4ca918332b08f)
- **Discussion:** [#78236](https://github.com/WordPress/gutenberg/pull/78236) · 11 comments · 0 reactions

## Summary

The experimental widget dashboard now renders a custom-styled resize handle for grid tiles instead of the default `@wordpress/grid` corner triangle. This UI enhancement improves the visual affordance and hit area for resizing, aligning the drag target with dashboard chrome styling while keeping hover, focus, and active states clearly visible. The change is strictly scoped to the experimental widget dashboard and does not alter core grid behavior or public APIs.

## Impact

- **Plugin & Theme Developers**: No action required. The default `@wordpress/grid` resize handle remains unchanged for other consumers.
- **Platform & Hosting Teams**: No action required. This is an internal UI update to the experimental widget dashboard route.
- **Site Owners**: No direct action required. Improved visual feedback when resizing tiles in edit mode.
- **Headless & REST Consumers**: No impact. This is a frontend-only rendering change with no API or schema modifications.

## Technical details

The diff introduces `WidgetResizeHandle` as a dedicated component, passing it via the existing `renderResizeHandle` prop on both branches of `DashboardGrid` in `widgets.tsx`:

```tsx
// Before (implicit default)
<DashboardGrid items={ items } />

// After (dashboard-specific handle injected)
<DashboardGrid 
  items={ items } 
  renderResizeHandle={ WidgetResizeHandle }
/>```

- `widget-resize-handle.tsx` implements `ResizeHandleRenderProps`, branching logic for `verticalResizable` tiles, and forwards `dnd-kit` refs/listeners/attributes.
- `widget-resize-handle.module.css` isolates all styling: an inset L-shaped stroke rendered on `::after`, token-driven spacing (`--widget-resize-handle-visual-inset`), scale transitions on hover/focus/active states, and a forced-colors fallback using `Highlight`.
- No REST routes, block.json schemas, PHP filters, or core grid algorithms were modified.

## Contribution

PR #78236 was opened by @jameskoster and merged as commit `71498dd` following review from @retrofox and @simison. The change addresses visual consistency in the experimental widget dashboard, aligning with related efforts (#77626, #77616). The implementation scope was kept strictly to the dashboard grid's resize affordance; no broader architectural changes or alternative handle strategies were merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
