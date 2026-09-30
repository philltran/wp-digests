# #81899: Grid: per-item tile size limits

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Grid`
- **Merged:** [`dad66c5`](https://github.com/WordPress/gutenberg/commit/dad66c5c37804413c4951deb93540b863a37f70f)
- **Discussion:** [#81899](https://github.com/WordPress/gutenberg/pull/81899) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`DashboardGrid` and `DashboardLanes` in `@wordpress/grid` gain an `itemLimits` prop that sets per-item minimum and maximum tile sizes in pixels, keyed by layout item key. Each surface converts the pixel limits into whole-track span bounds and enforces them on both rendered spans and the resize gesture. The limits are never written into the layout, so persisted data is untouched and a limit change re-renders every existing layout under the new rule.

## Impact

**Plugin/theme developers and hosts using `@wordpress/grid`**
- Additive, opt-in API. Omitting `itemLimits` leaves behavior unchanged, and the bounded layout keeps the source identity when no item needs bounding.
- Hosts can now declare floors and ceilings per tile (e.g. `chart: { minWidth: 320, minHeight: 200 }`) without polluting the stored layout.
- Stored spans outside the limits render bounded, but `onChangeLayout` still emits stored spans for untouched tiles. Consumers should not expect persisted data to reflect the bounded view.
- `'full'` width now respects a maximum (renders at the capped span), and `'fill'` reserves at least its minimum and never exceeds its maximum.
- `DashboardLanes` only applies the width axis (`GridItemWidthLimits`), since lane heights are content-driven. Height limits on `DashboardGrid` apply only when `rowHeight` is numeric.
- Translating widget-type declarations into limits in `@wordpress/widget-dashboard` is deferred to follow-up work (#81795).

Site owners and REST consumers: no impact.

## Technical details

**New API surface**
- `GridItemLimits` type: `{ minWidth, minHeight, maxWidth, maxHeight }` in pixels. The changelog also names a width-only `GridItemWidthLimits` for lanes.
- `itemLimits?: Record< string, GridItemLimits >` prop on `DashboardGrid`; `Record< string, GridItemWidthLimits >` on `DashboardLanes`.

```jsx
<DashboardGrid
	layout={ layout }
	rowHeight={ 80 }
	itemLimits={ {
		chart: { minWidth: 320, minHeight: 200 },
		note: { maxWidth: 500 },
	} }
>
	{ tiles }
</DashboardGrid>
```

**Implementation (from the diff, which is truncated)**
- `packages/grid/src/dashboard-grid/index.tsx` uses new `useSpanBounds()` and `useResizePixelLimits()` hooks from `shared/use-span-bounds`. Per the description, `pixelLimitsToSpanBounds()` does the quantization: minimums round up, maximums round down, width bounds saturate at the column count, and a minimum exceeding a maximum after quantization wins.
- `activeLayout` is now a `useMemo` that maps `temporaryLayout ?? layout` through `clampSpan()` (from `shared/resize-snap`) for numeric widths and heights. It returns the original array when nothing changes.
- Commit paths (`handleResize`/reorder) now build the updated layout from `latestLayoutRef.current ?? layout` (the stored layout) instead of the bounded `activeLayout`, so bounded spans never reach `onChangeLayout`.
- Resize: the new width and height are clamped with `clampSpan` against per-item bounds. For `'full'` items the baseline width is `bounds?.maxWidth ?? effectiveColumns`.
- `GridItem` gains `maxResizeWidthPx` and `maxResizeHeightPx` props, passed as a fourth argument to `clampResizeDelta`, which gains a `maxSize` parameter. The height max is only passed when `verticalResizable`.
- `resolveFillWidths()` in `resolve-fill-widths.ts` takes an optional `itemBounds` map (`{ minWidth, maxWidth }`). It plans rows around each fill's bounds, reserving at least the floor, wrapping to a wide-enough free run, and capping the result. A `'full'` item capped below the column count places like a fixed item.
- `GridItem`'s `maxColumns` is now `spanBoundsByKey.get( id )?.maxWidth ?? effectiveColumns`.
- The Lanes implementation and the stories/tests fall in the truncated part of the diff and are not verified here beyond the description.

Also updated: `packages/grid/README.md` (new prop rows and a "Size limits" section) and `CHANGELOG.md`. No PHP, REST, or DB changes.

## Contribution

Authored by @retrofox as part of the tracking issue #81795, with @chihsuan credited by the props bot. The PR was scoped to the grid-level capability only; the widget-type-to-limits translation was deliberately split into a separate follow-up. The discussion consists of bot comments only (props, size check, and a flaky navigation e2e test unrelated to the change), so no design debate is recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
