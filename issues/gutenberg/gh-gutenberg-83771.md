# #83771: UI: Recommend Checkbox family components

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Tool] ESLint plugin`, `[Package] UI`
- **Merged:** [`128b768`](https://github.com/WordPress/gutenberg/commit/128b76836d8ed4dc61a36a92b3ed3995d715db9e)
- **Discussion:** [#83771](https://github.com/WordPress/gutenberg/pull/83771) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`Checkbox`, `CheckboxControl`, and `CheckboxGroup` from `@wordpress/ui` are now marked **recommended** in Storybook and the component manifest. The legacy `CheckboxControl` from `@wordpress/components` is marked **not recommended**. The `@wordpress/use-recommended-components` ESLint rule now reports imports of the legacy control and points to a new Storybook migration guide. Existing legacy usages are recorded in `tools/eslint/suppressions.json`, so only new imports are flagged.

## Impact

- **Gutenberg / core contributors:** `npm run lint:js` now flags new imports of `CheckboxControl` from `@wordpress/components`. Existing callers are baselined in `tools/eslint/suppressions.json` (for example `block-lock/modal.jsx`, `block-manager/*`, `link-control/settings.jsx`, `image/image.jsx`). Use `CheckboxControl` from `@wordpress/ui` for new code.
- **Plugin and theme developers:** Nothing is removed or deprecated at runtime. `@wordpress/components` `CheckboxControl` still works and ships unchanged. The change affects guidance and lint only. Anyone using `@wordpress/eslint-plugin` with `use-recommended-components` enabled will see new reports on legacy `CheckboxControl` imports after upgrading.
- **Migrators:** The migration is not a drop-in swap. `onChange` becomes `onCheckedChange`, and `help` becomes `description`, or `details` when the help text contains markup. `label` is required and must be a plain string. A checked checkbox with `name` but no `value` submits `"on"` instead of `"1"`. Legacy `indeterminate` parent/child patterns map to `CheckboxGroup` with `allValues` and a `parent` checkbox.
- **Site owners, hosting, and REST consumers:** No action required.

## Technical details

**Storybook and manifest**
- `packages/ui/.../checkbox/`, `.../checkbox-group/`, and `.../checkbox-control/stories/index.story.tsx` gain `tags: ['manifest']` and `componentStatus: { status: 'recommended', whereUsed: 'global' }`. `Checkbox` and `CheckboxGroup` previously had `use-with-caution`, with notes about style consistency (issue #76135). Those notes are removed.
- `packages/components/src/checkbox-control/stories/index.story.tsx` drops the `manifest` tag and sets `status: 'not-recommended'`, with a note pointing to the `@wordpress/ui` control and the migration guide.
- New `packages/components/src/checkbox-control/stories/migration-guide.mdx` covers prop mapping, form values, labels, and `CheckboxGroup`.
- `storybook/components-manifest.yml` now lists `Checkbox`, `CheckboxControl`, and `CheckboxGroup` (with `CheckboxGroup.NestedItems`) from `@wordpress/ui`. The legacy props `__nextHasNoMarginBottom`, `heading`, `help`, and `onChange` are gone from the manifest. `CheckboxControl:heading` is removed from `prop-description-allowlist.json`.
- A JSDoc comment is added for `CheckboxGroupProps.children`.

**ESLint rule** (`packages/eslint-plugin/rules/use-recommended-components.js`)
```js
// ALLOWLIST['@wordpress/ui'] — added
'Checkbox', 'CheckboxControl', 'CheckboxGroup'
// (Checkbox and CheckboxGroup moved out of the previous list)

// DENYLIST['@wordpress/components'] — added
CheckboxControl:
  'Use `CheckboxControl` from `@wordpress/ui` instead. See migration guide in the lint rule documentation.'
```
`docs/rules/use-recommended-components.md` adds a migration-guide link to the Storybook page `components-checkboxcontrol--migration-guide`.

**Suppressions:** `tools/eslint/suppressions.json` gains or increments `@wordpress/use-recommended-components` counts across many `block-editor`, `block-library`, and `core-data` files. The diff is truncated, so the full list is not visible. CHANGELOG entries are added for `components`, `ui`, and `eslint-plugin`.

## Contribution

Authored by @mirka and merged with review input from @ciampo. The PR deliberately did not migrate existing callers. It baselines them in the ESLint suppressions file so new imports are caught without a large unrelated change. The record shows no notable design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
