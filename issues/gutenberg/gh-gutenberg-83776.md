# #83776: Add separators to item popup components

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`288b37e`](https://github.com/WordPress/gutenberg/commit/288b37e27b485ffdf91d390f9f0e33c2e667f065)
- **Discussion:** [#83776](https://github.com/WordPress/gutenberg/pull/83776) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/ui` gains a `Separator` subcomponent for the `Autocomplete`, `Combobox`, and `Select` primitives, re-exposed on `SearchableSelect` and `SearchableChipSelect` and (per the PR description and changelog) the corresponding select controls. Each one wraps the matching Base UI separator with a shared item-popup style, so popup choices or groups can be visually divided the way `Menu.Separator` already does for menus.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** New opt-in API; you can render `<Autocomplete.Separator />`, `<Combobox.Separator />`, `<Select.Separator />`, `<SearchableSelect.Separator />`, and `<SearchableChipSelect.Separator />` inside item popups. Existing usage is unchanged.
- **Everyone else:** No action required. There are no deprecations, removals, or migrations.
- **Bundle size:** The PR bot reports roughly +1.42 kB total across the `block-editor`, `editor`, and `workflow` bundles.

## Technical details

The diff adds a `separator.tsx` to each of `form/primitives/autocomplete`, `combobox`, and `select`. Each is a `forwardRef<HTMLDivElement, …>` wrapper around `_Autocomplete.Separator`, `_Combobox.Separator`, or `_Select.Separator` from `@base-ui/react`. It merges `itemPopupStyles.separator` (from `utils/css/item-popup.module.css`) into `className` and forwards the ref.

The `orientation` prop is destructured away and discarded, and new types (`AutocompleteSeparatorProps`, `ComboboxSeparatorProps`, `SelectSeparatorProps`) are defined as `Omit<ComponentProps<…>, 'orientation'>`. The orientation is therefore not configurable by consumers.

Each primitive's `index.ts` exports `Separator`. `SearchableSelect` and `SearchableChipSelect` attach the Combobox `Separator` via `Object.assign`, alongside `Group` and `GroupLabel`.

Usage pattern from the stories, which render the separator before a given item inside the collection render function:

```tsx
<Fragment key={ item.value }>
	{ item.value === 'other' && index > 0 && <Select.Separator /> }
	<Select.Item value={ item }>…</Select.Item>
</Fragment>
```

The CSS module change itself (thickness, spacing, color, forced-colors treatment) and the select-control wiring fall in the truncated part of the diff, so they are taken from the PR description rather than verified. Also added: `WithSeparator` stories, Storybook `subcomponents` metadata, ref-forwarding assertions in the Autocomplete and Combobox browser tests, and a `packages/ui/CHANGELOG.md` entry under New Features.

## Contribution

Authored by @mirka, with @ciampo credited in the props list. The record shows only a bot-generated metadata comment and the props list, with no design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
