# #82566: Searchable selects: Hide creatable footer when it is filtered out

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Type] Breaking Change`, `[Package] UI`
- **Merged:** [`234d5f9`](https://github.com/WordPress/gutenberg/commit/234d5f9508f7f461de684e973b8aac948b38c292)
- **Discussion:** [#82566](https://github.com/WordPress/gutenberg/pull/82566) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`SearchableSelect` and `SearchableChipSelect` (and `SearchableChipSelectControl`) in `@wordpress/ui` now render the `creatable: true` footer item only when that item survives Base UI's filtering. Previously the footer always mounted from the original `items` prop, so after an unmatched query the Create row was still clickable but unreachable with Arrow Down. The components no longer clone or rewrite the filter and item groups to pin Create back into the keyboard list; keeping the creatable item in the filtered set is now the consumer's responsibility.

## Impact

- **Plugin/theme/Gutenberg-package developers using `@wordpress/ui` searchable selects (labeled Breaking Change):**
  - If your creatable item has a static label (e.g. always "Create new item"), it will now disappear once the query doesn't match it. Previously the footer stayed visible. To keep it, give the item a label that tracks the query (e.g. "Create new item: apple") or use a `filter` that matches it.
  - Groups that mix `creatable: true` items with regular items now trigger a dev warning. Put the creatable item in its own group.
  - Public props are unchanged; no API renames or removals on the public surface.
- **Site owners / REST / hosting:** No action required.

## Technical details

- `SearchableChipSelect` now passes `items` straight to `Combobox.Root` (previously `normalizeRootItems( items )`, which reordered flat lists to put creatable items last and rewrote grouped lists to append a creatable-only group).
- The popup body is extracted into a new shared component, `packages/ui/src/form/primitives/searchable-results.tsx`. It reads `filteredItems` inside Root (via Base UI's `Combobox`), renders the creatable item in `Combobox.ListFooter` only if it's still in the filtered collection, and skips creatable entries in the `Combobox.Collection` body (including groups that contain only creatable items).
- Removed from `searchable-chip-select/types.ts`: `isItem`, `findCreatableItem`, `shouldSkipCollectionEntry`, `normalizeRootItems`. `findCreatableItems` and `hasGroupedItems` now accept `ReadonlyArray< Item | ItemGroup >`.
- `dev-warnings.ts` adds a `warning()` when a group contains both creatable and non-creatable items: ``SearchableChipSelect: do not mix `creatable: true` items with regular items in the same group. Put the creatable item in its own group.``
- Tests added/changed: footer hidden on no-match and on partial match excluding the creatable item; footer kept when the query matches it (flat and grouped); keyboard selection of the footer when it is not last in a flat list; the mixed-group warning.
- Docs/JSDoc updated: `creatable` now documented as rendering in the footer "when it is in the filtered items", with creation handled in `onValueChange`.

```tsx
// Keep the create row visible by making its label track the query
const items = [
  ...options,
  { value: '__create__', label: `Create new item: ${ query }`, creatable: true },
];
```

The diff is truncated, so the `SearchableSelect` primitive's own changes are not visible; the `SearchableChipSelect` path is what is confirmed above.

## Contribution

Opened by @mirka as a follow-up to review feedback on #80961 and as an alternative to #82462, which kept the footer visible by cloning Base UI's filter and pinning Create onto the keyboard list. That approach was rejected as "more magic than our components should own". In review, @mirka noted the original bug only surfaces when the creatable label is static rather than query-dependent, which is why it escaped earlier testing in #80967. @Mamaduka flagged a `packages/ui/CHANGELOG.md` merge conflict before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
