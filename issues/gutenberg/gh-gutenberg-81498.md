# #81498: DataViews: Fix date and datetime controls selecting the adjacent day

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`87702de`](https://github.com/WordPress/gutenberg/commit/87702decc4bcb5e5ecaf641290e04d4d3fc28c7a)
- **Discussion:** [#81498](https://github.com/WordPress/gutenberg/pull/81498) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The DataViews `date` and `datetime` DataForm controls no longer select or highlight the day adjacent to the one clicked when the site timezone differs from the browser's. The calendar reports and reads `Date` values in its `timeZone` prop (or the browser zone when absent), and the controls mixed those frames, so the stored value and the visible selection could be off by one day. The fix treats plain `date` values as browser-local calendar days and gives the `datetime` calendar the site's timezone, including manual UTC offsets.

## Impact

- **Site owners / editors:** With a site timezone that differs from the browser's (especially a manual UTC offset rather than a named zone), clicking a day in a DataForm date or datetime field now keeps that day selected and shows it in the input. An empty `datetime` now starts at `00:00` in the site timezone instead of the current time of day.
- **Plugin & theme developers using `@wordpress/dataviews`:** No API change and no action required. If you have tests or logic relying on the old off-by-one behavior, or on the previous `timeZone` passed to the `date` calendar, re-check them.
- **Known limitation:** Plain `date` fields use the browser's civil calendar. A civil day the browser timezone historically skipped (e.g. `2011-12-30` in `Pacific/Apia`) cannot round-trip exactly. The authors deliberately left this edge unaddressed.
- **Test environments:** Raw UTC-offset timezone identifiers are accepted by target browsers and Node 22+, but Node 20's `Intl.DateTimeFormat` rejects them. The manual-offset integration tests therefore only run where offset identifiers are supported.

## Technical details

Changes are in `packages/dataviews/src/components/dataform-controls/date.tsx` and `datetime.tsx`, with tests and a CHANGELOG entry.

**`date.tsx`**
- `parseDate` now uses `date-fns` `parseISO` instead of `getDate`, so a `yyyy-MM-dd` string is parsed in the browser timezone, the same frame the calendar reads it in.
- All `toCalendarDate( …, timezoneString )` conversions and the `timezoneString` dependency are removed from `CalendarDateControl` and `CalendarDateRangeControl`.
- The `timeZone` prop is no longer passed to `Calendar` or `RangeCalendar`.
- `onSelectDate` uses `formatDate( newDate )` rather than an inline `format( newDate, 'yyyy-MM-dd' )`.
- Month-sync effects (from #81635) are simplified to compare dates directly with `isSameMonth` and `areIntervalsOverlapping`, since all dates are now in one frame.

**`datetime.tsx`**
- The calendar's timezone is now `const timeZone = timezone.string || dateI18n( 'P' );`, i.e. the named site timezone, or the localized `±HH:MM` offset for sites on a manual offset.
- `timeZone` is passed to `Calendar` and used in `toCalendarDate` and the month-sync effect, so the selected day and Calendar's built-in "today" marker follow the site timezone.
- In `onSelectDate`, a new value without an existing time now uses `'00:00'` (site midnight) instead of `dateI18n( 'H:i', newDate )`.

```tsx
// before (datetime)
timeZone={ timezoneString || undefined }
// after
const timeZone = timezone.string || dateI18n( 'P' );
timeZone={ timeZone }
```

No new Calendar API is added; Calendar's existing timezone handling derives the selected day and today. Added tests cover clicked-day round trips, selected-day highlighting, empty datetimes at midnight, ranges, named and manual timezones, and external value changes. The diff was truncated, so the remaining test details are not reviewed here.

## Contribution

This PR took over from an earlier attempt (#81350) as an alternative approach, and @ciampo asked to close that one and continue here. The first version was closed by @ntsekouras after CI showed passing site UTC offsets as the calendar `timeZone` crashed in Node 20, because `Calendar` builds `Intl.DateTimeFormat`s from that prop in `use-localization-props.ts`; @ciampo did not consider lack of Node `Intl` support a blocker. @ciampo then pushed commits to address feedback and pre-approved it, and stated the merge order: this PR, then #81814 (locale-string support in Calendar), then #81592 (adopting that API in DataViews while preserving this fix). The description notes the PR was originally AI-generated and then reviewed and adjusted manually.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
