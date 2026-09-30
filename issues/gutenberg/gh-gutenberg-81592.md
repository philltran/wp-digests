# #81592: DataViews: Pass the WordPress locale to the date/datetime calendar controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @i-am-chitti
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`
- **Merged:** [`194f70e`](https://github.com/WordPress/gutenberg/commit/194f70eaa2aab91326a5824af511d8237a3e8a0a)
- **Discussion:** [#81592](https://github.com/WordPress/gutenberg/pull/81592) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `date` and `datetime` DataForm controls (and the `RangeCalendar` used for date ranges) now pass the site's locale and text direction to the `@wordpress/ui` `Calendar`/`RangeCalendar` components. Previously no `locale` was passed, so month and weekday names always rendered in English regardless of site language. A new `getCalendarLocale()` utility converts WordPress locale slugs into BCP 47 tags, and `dir` is derived from `isRTL()` so RTL admins get an RTL calendar.

## Impact

- **Site owners / end users:** Calendars in DataViews/DataForm (e.g. Site Editor → Pages → Add filter → Date) now show localized month headers and weekday abbreviations, and flip layout in RTL languages.
- **Plugin & theme developers:** No API change and no action required. Any DataForm using the `date` or `datetime` controls picks this up automatically. If you have visual or e2e tests that assert English calendar labels under a non-English locale, they may need updating.
- **Hosting / headless / REST:** Not affected.

## Technical details

**New utility:** `packages/dataviews/src/components/dataform-controls/utils/get-calendar-locale.ts` exports a default `getCalendarLocale( wpLocale: string ): string | undefined`.

- Splits the slug on `_` or `-`, lowercases the language, and applies `LANGUAGE_ALIASES` (`ary` → `ar-MA`, `haz` → `fa`).
- Keeps only subtags that look like a script (4 letters) or region (2 letters or 3 digits). Variant suffixes such as `formal` or `ao90` are dropped, so `de_DE_formal` → `de-DE` and `pt_PT_ao90` → `pt-PT`. Per the PR discussion, this also avoids a latent `RangeError` from `Intl` on invalid variants.
- Returns `undefined` for an empty or missing slug, in which case the calendar uses its default.

**Wiring:** In `date.tsx` (both `CalendarDateControl` and `CalendarDateRangeControl`) and `datetime.tsx` (`CalendarDateTimeControl`):

```tsx
const locale = getCalendarLocale( getSettings().l10n.locale );
// ...
<Calendar
  locale={ locale }
  dir={ isRTL() ? 'rtl' : 'ltr' }
  weekStartsOn={ weekStartsOn }
  ...
/>
```

`dir` comes from `isRTL()` (`@wordpress/i18n`) rather than the locale. This means an RTL site still gets an RTL calendar when `Intl` has no data for the language and the calendar falls back to `en-US` (e.g. `skr`).

**Tests:** Adds unit tests for `getCalendarLocale` and two `DateControl` localization tests (Urdu month header rendering; `dir="rtl"` for `skr`). A CHANGELOG entry is added to `packages/dataviews`.

## Contribution

The first iteration imported `date-fns` locale data, which reviewer @ciampo flagged for bundle size. The author reworked it to import only `enUS` and cache a copy carrying the normalized code, then, after @ciampo's follow-up PR #81814 (Calendar improvements including a Gregorian calendar bug fix) merged, rebased and switched to passing a plain BCP 47 string to `locale`. The final implementation avoids `date-fns` locale data entirely. The author noted that `haz` → `fa` is not an official alias, just the nearest language in the same script. The PR disclosed use of an AI tool for the implementation plan and tests.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
