# #80574: Combobox, Select, SelectControl: Add Group and GroupLabel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`b4874cd`](https://github.com/WordPress/gutenberg/commit/b4874cdbb86fbc5ccc1baeb3fa6f03164431c7eb)
- **Discussion:** [#80574](https://github.com/WordPress/gutenberg/pull/80574) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds `Group` and `GroupLabel` subcomponents to the `Combobox`, `Select`, and `SelectControl` UI primitives, enabling developers to render labeled option groups. This brings parity with the existing `Autocomplete` component, which already supported grouped items, and ensures consistent styling across the form primitive family.

## Impact

- **Plugin & theme developers**: Can now compose grouped dropdowns and comboboxes with labeled sections using `Combobox.Group`, `Select.Group`, or `SelectControl.Group` alongside their respective `GroupLabel` components.
- **Site owners**: No direct impact; behavior is isolated to the block editor and admin UI components.
- **No breaking changes or deprecations**. Existing codebases require no migration or configuration updates.

## Technical details

The diff introduces `group.tsx` and `group-label.tsx` wrappers in `packages/ui/src/form/primitives/combobox/` and `packages/ui/src/form/primitives/select/`. Each `Group` component uses `forwardRef` to pass props to `@base-ui/react/combobox.Group` or `@base-ui/react/select.Group`, applying `itemPopupStyles.group` for spacing. `GroupLabel` wraps the corresponding `@base-ui/react/...GroupLabel` inside a `Text` component with `variant="heading-sm"` and applies `itemPopupStyles[ 'group-label' ]`. The `select-control/index.ts` file re-exports these primitives and assigns `displayName` properties (`SelectControl.Group`, `SelectControl.GroupLabel`) to attach them to the `SelectControl` object. Type definitions (`ComboboxGroupProps`, `SelectGroupProps`, `ComboboxGroupLabelProps`, `SelectGroupLabelProps`) are added to their respective `types.ts` files. Unit tests verify ref forwarding for grouped items, and Storybook fixtures demonstrate the nested `Collection` pattern required to iterate over groups and items.

## Contribution

Opened by @mirka and merged with co-authorship from @aduth, the implementation closely mirrors the existing `Autocomplete` group pattern to maintain consistency across the UI library. The PR included Storybook examples, type definitions, and ref-forwarding tests, with a straightforward review cycle resulting in a single merged commit.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
