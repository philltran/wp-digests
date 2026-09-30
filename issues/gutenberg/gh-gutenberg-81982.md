# #81982: UI: Set Calendar text direction automatically

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`8b13d25`](https://github.com/WordPress/gutenberg/commit/8b13d255d09bac49a273b087f43fb022f4580c00)
- **Discussion:** [#81982](https://github.com/WordPress/gutenberg/pull/81982) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`Calendar` and `RangeCalendar` in `@wordpress/ui` now compute their default text direction automatically. When `locale` is omitted, invalid, or unsupported, `dir` follows the WordPress text direction (`isRTL()` from `@wordpress/i18n`) instead of the `en-US` fallback, which previously forced LTR on RTL sites. Date formatting still falls back to `en-US`. An explicit `dir` prop still wins, and a supported `locale` still determines direction.

## Impact

- **Plugin & theme developers using `@wordpress/ui` calendars:** On RTL sites, calendars that omit `locale` (or pass an unsupported one such as `skr` on Node 20) now render RTL instead of LTR. Pass `dir` explicitly if you need the old behavior.
- **DataViews consumers:** No change. DataViews already passes an explicit `dir` independent of its date-formatting locale, and a new test locks this in.
- **Site owners:** RTL-language sites get correctly oriented date pickers where the locale was missing or unsupported. No action required.
- No deprecated or removed APIs.

## Technical details

Precedence for the resulting `dir`: explicit `dir` prop, then a supported `locale` (string or date-fns `locale.code`), then `isRTL()` from `@wordpress/i18n`.

- `calendar.tsx` / `range-calendar.tsx`: dropped the `locale = enUS` default (and the `@daypicker/react/locale` import), so `locale` is now `undefined` when omitted.
- `utils/use-localization-props.ts`: the `locale` param type changes from `NonNullable<BaseProps['locale']>` to `BaseProps['locale']`. The date-fns locale still falls back to `enUS` when `locale` is a string or `undefined`. A new `isRightToLeft` value is `isLocaleRTL( intlLocale )` when `supportedLocaleCode !== undefined`, else `isRTL()`. The returned `dir` uses it.
- `isLocaleRTL` now also reads the legacy `Intl.Locale#textInfo` property when `getTextInfo()` is unavailable. The type `IntlLocaleWithWeekInfo` is renamed `IntlLocaleWithInfo` and gains `textInfo`.
- `lang` still derives from the supported locale code, or `en-US`.
- Tests cover RTL/LTR site direction with no locale, supported RTL and LTR locales overriding site direction, the `skr` fallback, explicit `dir` override, and legacy `textInfo` for `sd` (rtl) and `ug-Latn` (ltr). A DataViews test confirms that site RTL wins over a supported `en` date locale.
- Docs (`usage-guidelines.mdx`) and the `packages/ui` CHANGELOG are updated.

## Contribution

This is a follow-up to #81814 by @ciampo. Auto-merge was disabled mid-review by @ntsekouras over a review comment, after which the PR was merged. The discussion record does not include the content of that thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
