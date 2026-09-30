# #81814: UI: Accept locale codes in Calendar components

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`75768b4`](https://github.com/WordPress/gutenberg/commit/75768b4e6fa8fc0e22c287b6a28ac8f7994f4a22)
- **Discussion:** [#81814](https://github.com/WordPress/gutenberg/pull/81814) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `Calendar` and `RangeCalendar` components in `@wordpress/ui` now accept a BCP 47 locale string (e.g. `"fr-FR"`, `"fa-IR"`) in the existing `locale` prop, in addition to a date-fns `Locale` object. Consumers no longer need to import a date-fns locale module just to localize date text. The PR also forces all `Intl.DateTimeFormat` instances to the Gregorian calendar so captions and accessible labels match the rendered grid (previously e.g. Persian could default to a non-Gregorian calendar), and the default first day of the week is now derived from browser `Intl` data for both input forms.

## Impact

**Plugin & theme developers using `@wordpress/ui` Calendar / RangeCalendar**
- New capability: `locale` is now typed `Locale | string`. No action required to keep existing code working.
- **Behavior change for date-fns locale objects:** if the locale object's `code` is supported by the browser, the default week start now comes from `Intl` week info rather than the date-fns locale's `options.weekStartsOn`. The changelog states existing object-based calendars may get a different default; pass `weekStartsOn` explicitly to preserve the old value.
- Non-Gregorian-default locales (e.g. `fa-IR`) now render Gregorian captions, numerals-in-Gregorian labels and accessible names; previously these could disagree with the grid.
- The navigation label (`labelNav`) is now translatable via `@wordpress/i18n` (`'Navigation bar'`) rather than leaking the English fallback.
- Explicit `weekStartsOn`, `dir` and `lang` props still take precedence over locale-derived values.

**Others**
- No REST, PHP, or hosting impact.

## Technical details

Changes are in `packages/ui/src/calendar/`, mainly `utils/use-localization-props.ts` and `types.ts`.

- `types.ts`: `locale?: Locale | string`; docs updated for `weekStartsOn` default ("based on the `locale` prop when available").
- `useLocalizationProps`:
  - If `locale` is a string, the date-fns locale passed to `DayPicker` is `enUS` (imported from `date-fns/locale`); Intl handles date text, numerals, direction and language. This is intentional (per discussion) to avoid bundling all date-fns locales; `enUS` remains the DayPicker fallback for the date-fns object.
  - `getSupportedLocaleCode()` uses `Intl.DateTimeFormat.supportedLocalesOf()` inside try/catch; invalid or unsupported codes fall back to `en-US`.
  - `getWeekStartsOn()` reads `Intl.Locale#getWeekInfo()` or legacy `weekInfo` and maps `firstDay` (1–7) to 0–6 via `% 7`. If unavailable, the existing default is used. For date-fns locale objects with an unsupported code, the object's own `weekStartsOn` is kept.
  - `isLocaleRTL()` now takes an `Intl.Locale` instead of a string.
  - All `Intl.DateTimeFormat` formatters (month name, narrow/long weekday, full date, and others) now pass `calendar: 'gregory'`.
- Tests (`test/localization.test.tsx`, `test/__utils__`) cover string locales, Persian Gregorian output, RTL, invalid/unsupported codes, legacy `weekInfo`, missing week info, and override precedence.
- Storybook adds a "Persian (locale code)" option; usage guidelines and CHANGELOG updated.

```jsx
// Before
import { fr } from 'date-fns/locale';
<Calendar locale={ fr } />

// After (also still valid)
<Calendar locale="fr-FR" />
```

The diff shown was truncated, so remaining implementation details (e.g. the `labelNav` wiring and the tail of the hook) are not reviewed here.

## Contribution

Authored by @ciampo as a follow-up related to #81592, with OpenAI Codex disclosed as used for implementation, tests and docs. @jsnajdr's review raised whether passing `enUS` to `DayPicker` for string locales was acceptable; @ciampo confirmed it is deliberate (avoid bundling date-fns locales) and pushed an additional fix routing `labelNav` through `@wordpress/i18n`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
