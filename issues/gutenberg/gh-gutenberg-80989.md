# #80989: SearchableChipSelect: Add grouped items support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Type] Breaking Change`, `[Package] UI`
- **Merged:** [`047dc27`](https://github.com/WordPress/gutenberg/commit/047dc276e29464758020f6b301dd9075c838476f)
- **Discussion:** [#80989](https://github.com/WordPress/gutenberg/pull/80989) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`SearchableChipSelect` in `@wordpress/ui` now supports grouped items. It exports `Group`, `GroupLabel`, and `Collection` subcomponents, and the default collection renderer now passes custom `children` straight through to `Combobox.Collection`. The `creatableItem` prop is removed. A creatable footer action is now an entry in `items` marked `creatable: true`, which the component pulls out of the main list and renders in the footer. Development warnings were added for common misconfigurations.

## Impact

**Plugin/theme developers using `@wordpress/ui`**
- **Breaking:** the `creatableItem` prop on `SearchableChipSelect` is removed. Move the item into `items` with `creatable: true` and handle selection in `onValueChange` as before.
- Grouped `items` (an array of `{ label, items }`) require a `children` renderer. There is no default renderer for groups.
- New exports: `SearchableChipSelect.Group`, `SearchableChipSelect.GroupLabel`, `SearchableChipSelect.Collection`.
- Dev-only warnings now fire for misconfigured `items`/`children`.
- `@wordpress/ui` gains a dependency on `@wordpress/warning`.

**Site owners / hosting / REST consumers:** No action required.

## Technical details

**Component (`searchable-chip-select.tsx`)**
- `creatableItem` is no longer destructured from props. It is derived with `findCreatableItem( items )`, and `normalizeRootItems( items )` produces the list passed to `Combobox.Root`. Per the PR description, this moves the creatable item to the end of flat `items` so keyboard order puts it last.
- The `Combobox.Collection` callback now receives `entry: Item | ItemGroup`. `shouldSkipCollectionEntry( entry, creatableItem )` returns `null` for the creatable entry. If `children` is given it is called with the entry. Otherwise non-item entries (groups) render `null`, and items render the default `Combobox.Item`.
- `index.ts` sets `displayName` on `Group` and `GroupLabel` and exports them and `Collection` from `../combobox/*` via `Object.assign`.
- The helpers `isItem`, `isItemGroup`, `findCreatableItems`, `hasGroupedItems` and the `ItemGroup` type live in `types.ts`. That file's diff is truncated, so their exact implementations are not visible here.

**Dev warnings (`dev-warnings.ts`)**
`warnSearchableChipSelectProps( items, children )` uses `@wordpress/warning` to warn on:
- more than one `creatable: true` item;
- grouped `items` with no `children`.

The description also lists warnings for a creatable item that is not last in flat lists and for a creatable item mixed with regular items in a group. Those checks are not in the visible part of the diff.

**Before/after**
```tsx
// Before
<SearchableChipSelect items={ ITEMS } creatableItem={ { value: 'create', label: 'Create new item' } } />

// After
<SearchableChipSelect items={ [ ...ITEMS, { value: '__create__', label: 'Create new item', creatable: true } ] } />
```

For grouped lists, the creatable item goes in its own group, e.g. `{ label: '', items: [ creatableItem ] }`, as in the `GroupedCreatable` story.

**Tests and stories**
- New `Grouped` and `GroupedCreatable` stories.
- The `Creatable` story is updated to the new API.
- Unit tests cover flat, grouped, and creatable rendering, plus the warnings, using a `jest.mock( '@wordpress/warning' )` mock.
- The CHANGELOG lists the removal under Breaking Changes.

## Contribution

Authored by @mirka and merged with review input credited to @ciampo and @Mamaduka. The visible discussion is only bot comments (bundle size, a flaky e2e test report, props list), so no design debate is recorded. The one visible design shift is replacing the `creatableItem` prop with an item-level `creatable: true` marker.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
