# #83493: UI: Add item labels and descriptions to remaining popups

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] UI`
- **Merged:** [`ebbb502`](https://github.com/WordPress/gutenberg/commit/ebbb50251d3e05984eceb53c97eae73311726bc2)
- **Discussion:** [#83493](https://github.com/WordPress/gutenberg/pull/83493) · 2 comments · 0 reactions
- **Usefulness:** 5/5

## Summary

The `@wordpress/ui` package now requires `ItemLabel` as the first direct child of `Autocomplete.Item`, `Combobox.Item`, `SearchableSelect.Item`, `SearchableChipSelect.Item`, `SearchableSelectControl.Item`, and `SearchableChipSelectControl.Item`, with optional `ItemDescription` children following. This extends the item-composition pattern previously introduced for `Select` and `SelectControl` (#82369) to the remaining popup components, giving every item a structured label/description split that feeds `aria-labelledby` and `aria-describedby` on the underlying option element. Passing raw text directly to these `Item` components is no longer valid and will throw in development.

## Impact

- **Plugin & theme developers using `@wordpress/ui` popups:** Breaking change. Any code that renders plain text (or arbitrary JSX) directly inside `Autocomplete.Item`, `Combobox.Item`, `SearchableSelect.Item`, `SearchableChipSelect.Item`, `SearchableSelectControl.Item`, or `SearchableChipSelectControl.Item` must wrap the primary text in `<ItemLabel>` and move supplementary text into `<ItemDescription>`. A dev-mode error is thrown if `ItemLabel` is not the first direct child or if `ItemDescription` is not a direct child of `Item`.
- **Searchable select/chip-select consumers:** Items now also accept an optional `description` string prop on the item value object, which is rendered as the item description automatically.
- **No action required** if you only use `Select` / `SelectControl` (already migrated in #82369) or do not use the affected popup components.
- **Bundle size:** +1.71 kB (+0.02%) across the editor and workflow bundles.

## Technical details

Each affected primitive gains two new exported subcomponents and a shared composition hook:

- `ItemLabel` — renders `<Text variant="body-md">` with the `item-label` CSS class. Serves as the option's accessible name.
- `ItemDescription` — renders `<Text variant="body-sm">` with the `item-description` CSS class. Contributes to the option's accessible description. Carries a `validationToken` (`ITEM_DESCRIPTION_DIRECT_CHILD` symbol) that throws in non-production builds if the component is not a direct child of `Item`.

The `Item` component in each primitive now calls `useItemContent(children, ITEM_CONTENT_COMPONENTS, existingAriaProps)` from `packages/ui/src/utils/item-popup`. This hook:

1. Validates that the first direct child is the `Label` component and all subsequent children are `Description` components.
2. Generates `aria-labelledby` (pointing at the label element) and `aria-describedby` (pointing at description elements) IDs.
3. Returns the validated children plus the computed ARIA props.

The item's rendered output wraps the content in a `<div className={itemPopupStyles['item-text']}>` and spreads the computed `itemAriaProps` onto the underlying Base UI item element.

Before (any of the six affected components):
```jsx
<Autocomplete.Item value={item}>
  {item.value}
</Autocomplete.Item>
```

After:
```jsx
<Autocomplete.Item value={item}>
  <Autocomplete.ItemLabel>{item.value}</Autocomplete.ItemLabel>
  <Autocomplete.ItemDescription>Suggested URL</Autocomplete.ItemDescription>
</Autocomplete.Item>
```

For searchable select/chip-select, the item value object can include a `description` string that is rendered automatically without explicit `ItemDescription` markup.

New exports per primitive (e.g. `packages/ui/src/form/primitives/autocomplete/index.ts`):
```ts
export { ItemDescription } from './item-description';
export { ItemLabel } from './item-label';
```

The `ITEM_CONTENT_COMPONENTS` object passed to `useItemContent` includes a `validationMessage` string (e.g. `'Autocomplete.ItemLabel must be the first direct child of every autocomplete item, followed only by Autocomplete.ItemDescription components.'`) used in dev-mode error messages.

## Contribution

Opened by @mirka as a follow-up to #82369 (which introduced the same pattern for `Select` and `SelectControl`). Co-authored with @ciampo. The PR carries only two comments (the automated bot meta and the co-author credit), with no visible design debate or rejected alternatives in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
