# #77562: Add `@wordpress/grid` package

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] New API`, `[Package] Grid`
- **Merged:** [`ae954bf`](https://github.com/WordPress/gutenberg/commit/ae954bf7bd988e8484b50f5895fc1567ac52fad5)
- **Discussion:** [#77562](https://github.com/WordPress/gutenberg/pull/77562) · 29 comments · 1 reactions

## Summary

Introduces the new `@wordpress/grid` package, providing a reusable 2D packed grid primitive (`DashboardGrid`) designed for dashboard-style surfaces. It standardizes drag-to-reorder and resize-handle behavior with explicit column/row spanning, removing the need for consumers to implement this layout logic atop `@dnd-kit` from scratch.

## Impact

- **Theme & plugin developers**: Gain a standardized, tree-shakeable component for building tile-based or dashboard UIs within the editor or frontend.
- **Gutenberg maintainers & ecosystem package authors**: Receive an official baseline for drag-and-drop grid layouts, reducing duplicated `@dnd-kit` implementations across packages.
- **Action required**: None. This is a net-new design-system package. Consumers must explicitly import `@wordpress/grid` to utilize the primitive.

## Technical details

- Adds `packages/grid/` scaffolding with core files `src/grid.tsx`, `src/grid-item.tsx`, `src/resize-handle.tsx`, and `src/resolve-fill-widths.ts`.
- Exports `DashboardGrid`, which consumes a `layout` array typed to `DashboardGridLayoutItem` (`{ key: string, width?: 'number' | 'fill' | 'full', height?: number, order?: number }`).
- Layout modes support fixed columns via the `columns` prop, or responsive column derivation via `ResizeObserver` when `minColumnWidth` is provided. The `width` property supports `'fill'` (consumes remaining row space) and `'full'` (spans all columns, forcing a row break).
- When `editMode` is enabled, the grid leverages `@dnd-kit/core` and `@dnd-kit/sortable`. It emits `onPreviewLayout` continuously during drag/resize gestures for live feedback, and fires `onChangeLayout` once on commit.
- The component uses `React.forwardRef` to attach refs to the root `<div>`. Props extend `React.ComponentPropsWithoutRef<'div'>`, allowing standard DOM attributes (`id`, `aria-*`, `data-*`, event handlers) to flow through while ensuring CSS Grid properties (`gridTemplateColumns`, etc.) are enforced by internal style modules.
- Aligns with WP design-system conventions: `package.json` sets `wpScript: false`, defines `build`/`build-module`/`build-types` exports, and scopes `sideEffects` to `src/**/*.module.css` for reliable tree-shaking.

## Contribution

Opened by @retrofox and merged as commit `ae954bf` after co-authoring/review contributions from @simison, @mirka, @fabiankaegy, @Mamaduka, @ciampo, @youknowriad, and @aduth. The final diff established the complete package structure, layout resolution logic, and dnd-kit integration. Follow-up items explicitly flagged as out-of-scope for this merge include keyboard-accessible resize handles, i18n localization of `@dnd-kit` screen-reader announcements, a potential compound-component API (`<DashboardGrid.Item>`), drag-drop compaction algorithms, and expanded test coverage for the public surface.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
