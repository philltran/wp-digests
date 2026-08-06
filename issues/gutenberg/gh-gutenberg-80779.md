# #80779: SearchableChipSelect: Add form primitive to @wordpress/ui

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`514d0f0`](https://github.com/WordPress/gutenberg/commit/514d0f0d1010298cfa7a219a7ecce36158e8879f)
- **Discussion:** [#80779](https://github.com/WordPress/gutenberg/pull/80779) · 10 comments · 4 reactions
- **Usefulness:** 4/5

## Summary

Adds a `SearchableChipSelect` form primitive to `@wordpress/ui` to provide a searchable multi-select control with chip-based selection state. The component composes existing `Combobox` primitives with `InputLayout` to reduce boilerplate for complex form controls in the block editor and admin UI.

## Impact

- **Plugin & theme developers** using `@wordpress/ui` gain a new primitive for searchable multi-select controls.
- **No breaking changes or deprecations** are introduced.
- Developers should note the component's Storybook status is set to `use-with-caution`, indicating pending style consistency reviews with `@wordpress/components`.
- No migration or configuration changes required.

## Technical details

The diff introduces `packages/ui/src/form/primitives/searchable-chip-select/` containing `searchable-chip-select.tsx`, `types.ts`, `style.module.css`, and Storybook/test fixtures. It exports `SearchableChipSelect` from `packages/ui/src/form/primitives/index.ts`. The component wraps `Combobox.Root` with `multiple` enabled, composing `Combobox.InputGroup`, `Combobox.Chips`, `Combobox.Value`, `Combobox.Input`, `Combobox.Popup`, `Combobox.List`, `Combobox.Collection`, `Combobox.Item`, and `Combobox.Clear`. It accepts `items`, `creatableItem`, `chipsContent`, `emptyContent`, `showClearButton`, `clearButtonLabel`, and `searchPlaceholder` props. CSS updates in `packages/ui/src/form/primitives/combobox/style.module.css` apply `max-width: 100%`, `line-height: var(--wpds-typography-line-height-xs)`, `min-width: 0`, and `overflow-wrap: anywhere` to `.input-layout` and `.chip-content` to handle chip wrapping. Storybook metadata sets `componentStatus.status` to `use-with-caution`.

## Contribution

Review highlighted edge cases around long chip text truncation and popover overlay positioning when the dropdown appears above the input; both were deferred to follow-up PRs. Async loading state support was discussed but not implemented in this merge. The component was explicitly tagged `use-with-caution` in Storybook pending broader style consistency reviews with `@wordpress/components`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
