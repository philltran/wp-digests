# #81635: DataViews: Sync the displayed calendar month with external value changes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`5d097d5`](https://github.com/WordPress/gutenberg/commit/5d097d530069d95196a5b6eca5273d22db752464)
- **Discussion:** [#81635](https://github.com/WordPress/gutenberg/pull/81635) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The DataForm `date` (single and `between` range) and `datetime` controls now update the displayed calendar month when their value changes from outside the control, such as after undo, reset, or switching the edited item. Previously the month was seeded from the value only on mount, so the selected day could end up off-screen. The shared `@wordpress/ui` `Calendar` and `RangeCalendar` also now preserve keyboard focus when a controlled month change removes the focused day, working around an upstream react-day-picker bug that dropped focus to `document.body`.

## Impact

- **Plugin/theme developers using DataForm** (including the experimental DataForm-driven block inspector): the calendar now follows external value changes. No API change and no action required.
- **Consumers of `@wordpress/ui` `Calendar` / `RangeCalendar`**: controlled `month` changes no longer lose day focus. Focus that was outside the day grid is not moved. If the new month has no enabled day, focus falls back to another calendar control.
- **Dependencies**: `@wordpress/dataviews` gains a new dependency, `@date-fns/tz` (`^1.5.0`). Lockfile-managed setups will see it appear.
- **Site owners / editors**: after undoing a date change, the calendar returns to the restored date's month.

## Technical details

**DataViews controls** (`packages/dataviews/src/components/dataform-controls/`):

- `date.tsx` (`CalendarDateControl`) and `datetime.tsx` (`CalendarDateTimeControl`) add a `useEffect` keyed on `[ value, timezoneString ]`. It parses the value and calls `setCalendarMonth` only if the target month differs from the current month (`isSameMonth`), with both compared in the WordPress site time zone. A cleared value leaves the current month unchanged.
- `CalendarDateRangeControl` adds an effect keyed on the range endpoints (`fromValue`, `toValue`, `timezoneString`). It keeps the current view if the range overlaps the displayed month (`areIntervalsOverlapping`, inclusive) or, when only one endpoint is set, if that endpoint is in the displayed month. Otherwise it jumps to the `from` month (falling back to `to`). This prevents the view from jumping back when the user selects the end of a cross-month range.
- A new helper, `utils/to-calendar-date`, is used (via `@date-fns/tz`) to normalize dates to the site time zone. It is applied to the initial month state and to the preset and manual-input handlers. `timezoneString` is now read from `getSettings().timezone.string` earlier in each component, and the `timeZone` prop passed to the range calendar uses it.

**Shared UI**: per the PR description and the maintainer's comment, `Calendar` and `RangeCalendar` in `@wordpress/ui` use DayPicker's day-focus callbacks and its current grid focus target to restore focus after a controlled month change. That part of the diff was truncated, so the implementation details are not shown here.

**Tests**: new unit tests cover `date.tsx` and `datetime.tsx` (external value change, cleared value, `between` operator, cross-month range selection, and a `Pacific/Kiritimati` (UTC+14) site time zone case), plus the single and range Calendar focus behavior. A `dataviews` CHANGELOG bug-fix entry was added.

## Contribution

The PR fixes issue #81511. Midway through review, @ciampo pushed a commit directly to the branch that moved the focus handling out of the DataViews controls and into the shared `@wordpress/ui` Calendar components, since the problem affects every consumer. He also opened the parallel upstream react-day-picker issue. That left DataViews owning only value-to-month synchronization. @jorgefilipecosta accepted the changes, and @ntsekouras is credited by the props bot. The PR discloses that AI tools were used in its preparation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
