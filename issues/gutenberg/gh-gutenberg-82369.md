# #82369: UI: Add Select.ItemLabel and ItemDescription

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Package] UI`
- **Merged:** [`73e8c53`](https://github.com/WordPress/gutenberg/commit/73e8c53ecae011c6b86a17625bab61d1dbd9b823)
- **Discussion:** [#82369](https://github.com/WordPress/gutenberg/pull/82369) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/ui` gains `Select.ItemLabel` and `Select.ItemDescription` (and the same on `SelectControl`), bringing Select items in line with the existing `Menu` item child contract. Items can now show supplementary text beneath the label, and `SelectControl` `items` accept an optional `description` string. The change is breaking for custom `Select.Item` / `SelectControl.Item` children, which must now start with `ItemLabel`. The block editor's Position control (Sticky/Fixed) is migrated to the new `description` field.

## Impact

**Plugin & theme developers using `@wordpress/ui`**
- **Breaking:** any `Select.Item` or `SelectControl.Item` with custom children (a string or arbitrary markup) must now wrap the content in `Select.ItemLabel`, optionally followed by `Select.ItemDescription` components. The CHANGELOG lists this under Breaking Changes.
- `SelectControl` with the `items` array is unchanged unless you want descriptions; add `description` to an item to opt in.
- Items with no children (e.g. `<Select.Item value="Item 1" />`) are also affected; the updated tests now pass an `ItemLabel` child.

**Site owners / editor users**
- Position sidebar control (blocks with `position` support) shows Sticky/Fixed hints as descriptions under each label. The hint is now exposed via `aria-describedby` rather than hidden with `aria-hidden`.

**Others**
- No server-side, REST, or block.json changes.

## Technical details

**New components** (`packages/ui/src/form/primitives/select/`):
- `item-label.tsx`: `ItemLabel` renders `Text variant="body-md"` with `render={ <_Select.ItemText render={ render } /> }`, so Base UI treats it as the item text. Per the doc comment, the trigger label still comes from the selected item's `label` (via `items` or an object value) or from `Select.Trigger` children.
- `item-description.tsx`: `ItemDescription` renders `Text variant="body-sm"` with the shared `item-popup.module.css` `item-description` class.
- Both are exported from `select/index.ts`.

**`Select.Item` (`item.tsx`):**
- Now calls the shared `useItemContent( children, ITEM_CONTENT_COMPONENTS, { aria-describedby, aria-label, aria-labelledby } )` from `utils/item-popup`, the same helper pattern as Menu. It validates that `ItemLabel` is the first direct child and only `ItemDescription`s follow; the validation message is `Select.ItemLabel must be the first direct child of every select item, followed only by Select.ItemDescription components.`
- The returned `itemAriaProps` are spread on `_Select.Item`. Per the CHANGELOG, multiple descriptions contribute to `aria-describedby` in DOM order.
- The previous `<_Select.ItemText>{ children }</_Select.ItemText>` wrapper is replaced by `<div className={ itemPopupStyles[ 'item-text' ] }>{ contentChildren }</div>`.

**Block editor (`hooks/position.jsx`):**
- `DEFAULT_OPTION`, `STICKY_OPTION` and `FIXED_OPTION` drop `key`, and `hint` becomes `description`.
- The manual `options.map` building `SelectControl.Item` children (with `useId` hint IDs and `aria-hidden`) is removed; `SelectControl` now receives the options via `items` and renders self-closing.
- The `.block-editor-hooks__position-control-item-hint` rule is deleted from `block-hooks.scss`.

**Before/after for custom children:**
```jsx
// Before
<Select.Item value={ item }>{ item.label }</Select.Item>

// After
<Select.Item value={ item }>
  <Select.ItemLabel>{ item.label }</Select.ItemLabel>
  <Select.ItemDescription>{ item.description }</Select.ItemDescription>
</Select.Item>
```

Stories (new `WithItemDescription`) and jsdom tests are updated accordingly. The diff shown is truncated, so the `SelectControl` implementation and type changes (`types.ts`, `items[].description`) are not visible here.

## Contribution

Authored by @mirka as a follow-up to #81825, with @ciampo credited in the props list. The PR description notes the same ItemLabel/ItemDescription pattern will be added to the other item popup components in separate PRs. The recorded discussion is only bot output (CodeRabbit skipped review as a draft; bundle-size and performance reports), so no design debate is visible.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
