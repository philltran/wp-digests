# #80961: SearchableSelect: Add form primitive to @wordpress/ui

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`d8476a5`](https://github.com/WordPress/gutenberg/commit/d8476a522daa178eebcc4589b9a580a61649baea)
- **Discussion:** [#80961](https://github.com/WordPress/gutenberg/pull/80961) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a `SearchableSelect` form primitive to `@wordpress/ui`: a trigger-based, searchable single-select built on the package's `Combobox`. It renders a `Combobox.Trigger` that opens a popup containing a search input, an item list (flat or grouped), an empty state, and an optional footer item for creating new entries. It is exported from `packages/ui/src/form/primitives` with `Item`, `Group`, `GroupLabel`, and `Collection` subcomponents.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** A new component is available. Nothing existing changes, so no action is required unless you want a searchable single-select.
- **Status caveat:** The Storybook `componentStatus` is `use-with-caution`. It is not yet recommended for use alongside `@wordpress/components`, pending review of style consistency, overlay compatibility, and component-set completeness (tracked in WordPress/gutenberg#76135).
- **Not a labeled field yet:** This is a primitive only. The PR description says a labeled control wrapper can be added on top later, so callers must supply `aria-label` or `aria-labelledby` themselves.
- **Site owners / hosting / REST consumers:** Not affected.
- **Changelog note:** The diff also moves the `Breadcrumb` entry into the existing "New Features" section of `packages/ui/CHANGELOG.md`. The Breaking Changes entries (fixed popup anchor width, portal theme inheritance) were already there and are not part of this PR's code changes.

## Technical details

**New files under `packages/ui/src/form/primitives/searchable-select/`:** `searchable-select.tsx`, `item.tsx`, `index.ts`, `types.ts`, `style.module.css`, Storybook stories and fixtures, and a jsdom test file.

**Component shape.** `SearchableSelect` is a `forwardRef<HTMLButtonElement, SearchableSelectProps>`. The ref goes to `Combobox.Trigger`. It composes:

- `Combobox.Root` (called with `false` for the multiple generic, so single-select), receiving normalized `items` plus the remaining props via `...restProps`.
- `Combobox.Trigger`, with `placeholder`, `aria-label`, `aria-labelledby`, `aria-describedby`, and `triggerContent` as its children.
- `Combobox.Popup`, with `width={ popupWidth }` and `aria-label` / `aria-labelledby` (the popup is exposed as a `dialog` with the same name).
- `Combobox.Input`, whose `placeholder` and `aria-label` both come from `searchPlaceholder` (default `__( 'Search' )`). It is wrapped in a div styled with `wpds-dimension-padding-*` tokens inside the `wp-ui` CSS layer.
- `Combobox.Empty`, rendering `emptyContent` (default `__( 'No results found.' )`).
- `Combobox.List`, `Combobox.ListBody`, and `Combobox.Collection`.
- `Combobox.ListFooter`, holding a `Combobox.Item variant="creatable"` when a creatable item is present.

**Item model.** Items are `{ value, label, disabled?, creatable? }`. `items` may also be an array of groups (`{ label, items }`). `findCreatableItem` picks out the item with `creatable: true`, and `shouldSkipCollectionEntry` keeps it out of the main collection so it appears only in the footer.

**Rendering.** If a `children` render function is passed, it is called as `children( entry, ...args )` for every collection entry. Otherwise flat items render as `Combobox.Item` with `entry.label`. Grouped items have no default renderer.

**Dev warnings** (via `@wordpress/warning`):

- More than one item with `creatable: true`.
- Grouped `items` supplied without a `children` renderer.

**Usage:**

```tsx
<SearchableSelect aria-label="Fruit" items={ ITEMS } />

// Grouped
<SearchableSelect
	aria-label="Fruit"
	items={ GROUPED_ITEMS }
	children={ ( group ) => (
		<SearchableSelect.Group key={ group.label } items={ group.items }>
			<SearchableSelect.GroupLabel>{ group.label }</SearchableSelect.GroupLabel>
			<SearchableSelect.Collection>
				{ ( item ) => (
					<SearchableSelect.Item key={ item.value } value={ item }>
						{ item.label }
					</SearchableSelect.Item>
				) }
			</SearchableSelect.Collection>
		</SearchableSelect.Group>
	) }
/>
```

`index.ts` sets `displayName` on `Item`, `Group`, and `GroupLabel` and attaches `Item`, `Group`, `GroupLabel`, and `Collection` via `Object.assign`. `Group`, `GroupLabel`, and `Collection` are reused directly from `../combobox/`, and `Item` wraps `ComboboxItem`. The PR does not add a `SearchableSelect` export to any package-level entry point beyond `form/primitives/index.ts`.

## Contribution

Authored by @mirka, with @ciampo listed among the accounts that interacted with the PR. The visible discussion is automated (bundle size, performance, a flaky e2e report, and CodeRabbit, which rated merge risk low and noted two minor concerns about shared component metadata and avoidable filtering work). The record shows no substantive human design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
