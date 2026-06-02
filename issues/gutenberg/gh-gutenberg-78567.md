# #78567: Font Library: Fix Update button staying active when changes are reverted

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Feature] Font Library`
- **Merged:** [`83de183`](https://github.com/WordPress/gutenberg/commit/83de183583cb537d27d413c21eee2242f22cdb42)
- **Discussion:** [#78567](https://github.com/WordPress/gutenberg/pull/78567) · 4 comments · 0 reactions

## Summary

Fixes a bug in the Font Library modal (Appearance > Fonts) where the **Update** button stayed enabled after a font-variant selection was toggled and then reverted to the last-saved state, falsely signaling unsaved changes. The previous logic only checked whether an edits entry existed (`!! globalStyles.edits.settings.typography.fontFamilies`), which never cleared once a checkbox was touched. The fix compares the edited `fontFamilies` value against the saved value from `globalStyles.record`, so the button reflects whether the selection actually differs from what was persisted.

## Impact

- **Site owners / editors:** The **Update** button in the Font Library now correctly disables when font-variant selections match the last-saved state, removing the misleading prompt to save when nothing changed. No action required.
- **Plugin & theme developers:** No API, hook, or schema changes — this is an internal correction to the Font Library UI's dirty-state detection. No action required.

## Technical details

The enable/disable check for the **Update** button previously tested only for the presence of an edits entry:

```js
// Before
const canBeUpdated = !! globalStyles.edits.settings.typography.fontFamilies;
```

Once any variant checkbox was toggled, the edits entry was set and never removed, so `canBeUpdated` stayed `true` even after reverting to the original selection. The change instead compares the edited value against the saved one from `globalStyles.record`:

```js
// After (conceptual)
const canBeUpdated = useMemo(
  () => getFontFamiliesKey( editedFontFamilies ) !== getFontFamiliesKey( savedFontFamilies ),
  [ editedFontFamilies, savedFontFamilies ]
);
```

A new `getFontFamiliesKey()` helper normalizes font families and their faces before comparison, so re-toggling a variant doesn't leave the button enabled merely because array order changed. The comparison is wrapped in `useMemo` so it only recomputes when the edited or saved value changes. The build size delta is small (+96 B total, +95 B in `editor/index.min.js`), consistent with an isolated comparison-logic change.

## Contribution

Opened and authored by **@juanfra**, labeled `[Type] Enhancement` / `[Feature] Font Library`, and merged as commit `83de183`. The discussion was light (4 comments), primarily the automated bundle-size report; no significant design debate or rejected alternatives are recorded in the provided material.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
