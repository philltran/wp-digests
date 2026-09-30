# #82291: DataForm: Communicate the timezone in the datetime control

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`f635771`](https://github.com/WordPress/gutenberg/commit/f635771bbd73990dd4f9a3fd8be83c668bbe9db4)
- **Discussion:** [#82291](https://github.com/WordPress/gutenberg/pull/82291) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `datetime` control in `@wordpress/dataviews` now renders help text under the input, such as `Timezone: (CEST) Europe/Copenhagen`, when the site timezone differs from the browser's. Sites with a manual UTC offset see `Timezone: UTC+14`, and a UTC site viewed from a non-UTC browser sees `Timezone: Coordinated Universal Time`. Nothing renders when the two timezones match. This clarifies which timezone a date is being edited in, for example in the Date filter and Quick Edit.

## Impact

**Plugin & theme developers using DataForm / DataViews**
- Any `datetime` DataForm control gets the new help text automatically when the site and browser timezones differ. There is no opt-in or opt-out.
- Custom UI or e2e/snapshot tests that assert on the control's accessible description or DOM may see the extra text.
- No API changes, deprecations, or breaking changes.

**Site owners / editors**
- In the site editor (the Pages Date filter and Quick Edit's Date field), the timezone is now shown when the site's Settings > General timezone differs from the machine's.

**Hosting, headless, REST consumers**
- No action required.

## Technical details

The diff adds a new util, `packages/dataviews/src/components/dataform-controls/utils/get-timezone-description.ts`, and passes its result to the input in `datetime.tsx` via `description={ getTimezoneDescription() }` inside `CalendarDateTimeControl`.

`getTimezoneDescription()` reads `getSettings().timezone` from `@wordpress/date`:

- It compares `Number( timezone.offset )` with `-new Date().getTimezoneOffset() / 60`. If they are equal it returns `undefined`, so no description is rendered.
- `zoneAbbr` is `timezone.abbr` when it is non-empty and non-numeric. Otherwise it is `UTC{+|}{offsetFormatted}`.
- Underscores in `timezone.string` are replaced with spaces, so `America/Costa_Rica` becomes `America/Costa Rica`.
- If `timezone.string === 'UTC'`, the detail is the translatable `Coordinated Universal Time`.
- Otherwise, if the zone name is non-empty, the detail is `(ABBR) Zone Name`.
- Otherwise the detail is the bare abbreviation or offset.
- The result is wrapped in `sprintf( __( 'Timezone: %s' ), detail )`, with a translators comment.

Because the string is passed as the input's `description`, the tests assert it via `toHaveAccessibleDescription`. The matching-timezone check compares offsets only, so two different zones with the same current offset are treated as matching and show nothing.

New jsdom tests in `datetime.jsdom.test.tsx` cover four cases:
- matching timezones (no description)
- a named zone
- a manual offset, wrapped in `describeWithOffsetTimeZones` because Node 20 rejects raw offset identifiers
- a UTC site with a non-UTC visitor, created by mocking `Date.prototype.getTimezoneOffset`

A `CHANGELOG.md` Enhancements entry was added for `@wordpress/dataviews`.

## Contribution

The author first tried appending the timezone as an input suffix, but it didn't fit in narrow containers such as the filter dialog without truncation or widening the dialog, so the PR moved to help text below the input. The PR notes that the named-timezone text can wrap to two lines in the filter popover, and that a field with its own description still renders below the calendar (two lines in the `compact` variant). Dropping the abbreviation or showing only the offset was floated as an alternative. The PR carries the `Needs Design Feedback` label; @jasmussen and @oandregal approved the help-text approach, with the latter noting it can be revisited.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
