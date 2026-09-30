# #81354: UI: Collapse item-popup item sizing to default and small

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Type] Breaking Change`, `[Package] UI`
- **Merged:** [`1f61578`](https://github.com/WordPress/gutenberg/commit/1f6157810ccbc40beed4089c843bec96c6d779d0)
- **Discussion:** [#81354](https://github.com/WordPress/gutenberg/pull/81354) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The popup item styling shared by `Select`, `Combobox`, and `Autocomplete` in `@wordpress/ui` now defaults to a 32px row height (previously 40px), matching the compact menu density from #73429. The separate `'compact'` item size is removed because it is now the default, and the `size` prop is dropped from `Combobox.Item` and `SelectControl.Item`. `Select.Item` keeps `'default'` and `'small'`. The PR is labelled as a breaking change.

## Impact

**Plugin & theme developers using `@wordpress/ui`**
- **Breaking (TypeScript and runtime):** `Select.Item` no longer accepts `size="compact"`. Use the default (omit `size`) instead.
- **Breaking:** `Combobox.Item` and `SelectControl.Item` no longer have a `size` prop. Remove any `size` you pass; it is no longer in the type, so TypeScript will flag it.
- `SelectControl` item size is no longer derived from the trigger `size`. Items always use the default height, regardless of trigger size.
- Visual change with no code change: all popup items render 32px tall rather than 40px. Snapshot/visual-regression tests and any layout that assumed 40px rows will shift.
- Horizontal padding on default items is `--wpds-dimension-padding-md`, matching `Menu`.

**Site owners / end users**
- No action required. Select and combobox dropdowns built on `@wordpress/ui` look denser.

**Hosting & platform / headless**
- Not affected.

## Technical details

Changes are confined to `packages/ui`.

**CSS (`utils/css/item-popup.module.css`)**
- `.item` now sets `--wp-ui-popup-item-height: var(--wpds-dimension-size-md)` (was `--wpds-dimension-size-lg`).
- The `&.is-size-compact` block is removed.
- `&.is-size-small` is kept.
- `.empty:not(:empty)` min-height also drops from `size-lg` to `size-md`, so the empty-state row matches the new item height.

**Components**
- `select/item.tsx`: the class is now applied only when `size === 'small'`, as `itemPopupStyles['is-size-small']`, instead of `is-size-${size}`.
- `select/types.ts`: `SelectItemProps['size']` changes from `InputLayoutProps['size']` to `'default' | 'small'`.
- `combobox/item.tsx` and `combobox/types.ts`: the `size` prop and its `is-size-*` class are removed.
- `select-control/item.tsx`: adds `SelectControlItemProps = Omit<SelectItemProps, 'size'>` and becomes a thin pass-through to `Select.Item`.
- `select-control/context.tsx` is deleted (`SelectControlSizeContext`, `useSelectControlSizeContext`), and the provider is removed from `select-control.tsx`.

```tsx
// Before
<Select.Item value={ item } size="compact" />
<Combobox.Item value={ item } size="compact" />

// After
<Select.Item value={ item } />
<Combobox.Item value={ item } />
```

Stories for Select and Combobox `Compact` drop the item `size` prop, and `packages/ui/CHANGELOG.md` records the breaking changes and the enhancement.

## Contribution

Follow-up to #73429, which moved `@wordpress/components` menu items from 40px to 32px; design discussion on that PR agreed to extend the density to listbox-style popovers. Review on this PR was light: @jasmussen gave a thumbs-up and asked what feedback was useful beyond code review, and @mirka said a glance at the screenshots (showing the relationship between the search fields and items) was sufficient. @ciampo is also credited by the props bot.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
