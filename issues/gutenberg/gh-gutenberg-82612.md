# #82612: UI: Reset completed calendar ranges by default

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`de9b2a6`](https://github.com/WordPress/gutenberg/commit/de9b2a6f4b7550100e22577b89ce94fd950d5f66)
- **Discussion:** [#82612](https://github.com/WordPress/gutenberg/pull/82612) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`RangeCalendar` in `@wordpress/ui` now starts a new range when a user clicks a date after a range is already complete, instead of extending or shortening the existing range. A new `resetOnSelect` prop, defaulting to `true`, exposes React DayPicker's option of the same name. The hover preview now matches: it outlines only the hovered date, which will become the new start. Passing `resetOnSelect={ false }` restores the previous behavior.

## Impact

**Plugin & theme developers using `@wordpress/ui` `RangeCalendar`**
- This is listed under **Breaking Changes** in the `packages/ui` CHANGELOG. The default interaction changed with no code change on your side.
- After the first two clicks complete a range, a third click now yields `{ from: thirdDate, to: undefined }`. Previously it yielded an extended or adjusted range.
- Single-click selection also changed shape. `onValueChange` now receives `{ from: date, to: undefined }` for the first click instead of `{ from: date, to: date }`. Code that assumes `to` is always defined after the first click needs to handle `undefined`.
- To keep the old behavior, pass `resetOnSelect={ false }`.
- Tests that simulate range selection by clicking a third date, or that expect `to` to equal `from` after one click, will need updating.

**DataViews / DataForm consumers**
- The `DateControl` range test in `packages/dataviews` no longer needs the workaround second click on the new end date. Range editing in that control now starts a replacement range on the next click.

**Site owners / hosting / REST consumers**
- No action required.

## Technical details

Changes are in `packages/ui/src/calendar/`:

- `types.ts`: adds `resetOnSelect?: boolean` (`@default true`) to `RangeProps`. The doc comment says a click starts a new range if there is no current start date or the range is already complete. When `required` is `false`, clicking the same day of a single-day range clears the selection.
- `range-calendar.tsx`:
  - `RangeCalendar` destructures `resetOnSelect = true` and passes it to the range-mode `DayPicker`.
  - `usePreviewRange` takes `resetOnSelect` as a new argument and adds it to the `useMemo` dependency array. When `resetOnSelect && value.to` is truthy, it returns `{ from: hoveredDate, to: hoveredDate }` immediately, before the existing extend/shorten preview logic.
- Docs and manifest: `usage-guidelines.mdx` describes the new behavior, `storybook/components-manifest.yml` lists `resetOnSelect`, and the CHANGELOG has a Breaking Changes entry.

Before/after for a completed range, then a third click:

```tsx
// before: extends/adjusts the existing range
// onValueChange -> { from: today, to: dayAfterTomorrow }

// after (default): starts a new range
// onValueChange -> { from: dayAfterTomorrow, to: undefined }

<RangeCalendar resetOnSelect={ false } /> // restores previous behavior
```

Tests in `range-calendar.jsdom.test.tsx` were updated so the old extend/shorten/clear cases explicitly set `resetOnSelect={ false }`, and new `usePreviewRange` cases cover the default path. The `usePreviewRange` hook is exported from `range-calendar.tsx`, and its signature now includes `resetOnSelect`.

## Contribution

Authored by @ciampo to close #82609, with review feedback from @simison and @retrofox per the props-bot list. In the discussion, @ciampo noted the behavior comes from upstream React DayPicker, which defaults `resetOnSelect` to `false`. It was not exposed earlier because the option was added upstream after the initial `Calendar` and `RangeCalendar` implementation. He also said the hover preview should help set user expectations. The PR notes Codex was used for implementation, tests, and docs, with the author reviewing the diff and browser behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
