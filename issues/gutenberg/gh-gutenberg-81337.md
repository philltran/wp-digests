# #81337: UI: Add `Calendar` and `RangeCalendar`, moved from `components` private APIs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @youknowriad
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`637513a`](https://github.com/WordPress/gutenberg/commit/637513a31fa427d34978bfecdf3b18d55f68c6d9)
- **Discussion:** [#81337](https://github.com/WordPress/gutenberg/pull/81337) · 12 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The private `DateCalendar` and `DateRangeCalendar` components are removed from `@wordpress/components` and reintroduced as the public `Calendar` and `RangeCalendar` components in `@wordpress/ui`. They are rewritten to that package's conventions: `--wpds-*` design tokens, CSS modules in the `wp-ui` cascade layers, `render` prop and ref forwarding, and `Button`/`Icon` primitives for month navigation. The DataViews `date` and `datetime` controls now import them from `@wordpress/ui`, which removes the private-API unlock for the `date` control and leaves `datetime` with one unlock (`ValidatedInputControl`).

## Impact

**Plugin & theme developers**
- **Breaking (private API removal):** `DateCalendar`, `DateRangeCalendar` and `TZDate` are no longer exported from `@wordpress/components`. They were locked private APIs, so only code that unlocked them is affected. Replacement: `Calendar` / `RangeCalendar` from `@wordpress/ui`; `TZDate` now comes from `@date-fns/tz`.
- `@wordpress/ui` does not re-export `TZDate`. It exports components only.
- `@wordpress/ui` is `0.x` and documents itself as experimental. Both components are marked `use-with-caution` in Storybook, so the prop surface may still change.
- `react-day-picker` moves from `@wordpress/components` to `@wordpress/ui`, which also declares `date-fns` directly. Anything relying on the transitive dependency via components should depend on it explicitly.
- Visual changes: the selected day background changes from `#1e1e1e` to `#2d2d2d` and now has a real hover state. The disabled+selected day uses `#8d8d8d` on `#e6e6e6`. Weekday headers are regular weight instead of bold.

**Site owners / editor users**
- The calendar in the editor's date fields (post Summary, Quick Edit, DataViews date filters) is meant to behave as before, aside from the visual deltas above.

**Bundle size**
- The CI size report shows `components` shrinking by ~22 kB, while `block-editor`, `edit-site`, `editor` and `media-utils` grow by ~20-24 kB each (+62.2 kB total). Investigating the DataViews bundle increase is listed as an unassigned follow-up.

**Hosting & headless:** No action required.

## Technical details

**Moved and rewritten**
- `packages/components/src/calendar/date-calendar` and `date-range-calendar` (including their READMEs) are deleted, and their `docs/manifest.json` entries are removed. The new implementations live in `packages/ui/src/calendar`.
- The only coupling to `@wordpress/components` was `useControlledValue`. `react-day-picker` owns the DOM, so there were no components to swap out.
- Styles become `style.module.css`. The class map is passed to `react-day-picker`'s `classNames` prop, replacing the global BEM strings.
- Month navigation buttons render `Button` (`minimal` / `neutral` / `compact`) and chevrons render `Icon` with `@wordpress/icons`, via `react-day-picker`'s `components` overrides.
- The root uses `useRender`, wired through a context so the `components` object stays referentially stable. A new component type on each render would remount the calendar and drop focus.
- Token mappings are mostly value-exact (e.g. `$grid-unit-40` → `--wpds-dimension-size-md`, `$radius-small` → `--wpds-border-radius-sm`).

**Dependencies** (`package.json` / `package-lock.json`)
- Removed from `packages/components`: `react-day-picker`.
- Added to `packages/ui`: `react-day-picker ^9.14.0` and `date-fns ^4.4.0`.

**Tooling**
- `Calendar` and `RangeCalendar` are added to the `use-recommended-components` lint allowlist so the DataViews import passes.
- The READMEs become a Storybook `Best Practices` MDX page, and prop tables are generated from JSDoc.

**Tests**
- 127 tests in `packages/ui/src/calendar`: 115 ported and 12 new for `render` / ref forwarding.

**Migration**
```diff
-import { DateCalendar, TZDate } from '@wordpress/components'; // via unlock
+import { Calendar } from '@wordpress/ui';
+import { TZDate } from '@date-fns/tz';
```

The `packages/components/CHANGELOG.md` entry records the removal. The diff was truncated, so the remaining `packages/ui` and DataViews files are not reviewed here.

## Contribution

Opened by @youknowriad as an alternative to #81324, which proposed vendoring the calendars into `@wordpress/dataviews`. That approach was argued to lead to duplicated code, since DataViews could never export them. @mirka spotted the bold weekday-header issue in review, and @ciampo reviewed and pushed fixes himself to speed iteration. He also collected follow-ups: syncing the displayed month with external DataViews value changes, duplicate DataForm updates, passing the WordPress locale, Calendar root role verification, removing `color-mix()`, and upgrading to `react-day-picker` 10. Separately, a Quick Edit calendar misalignment and off-by-one date bug, present on trunk too, was found and handed to @ntsekouras as follow-up issues.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
