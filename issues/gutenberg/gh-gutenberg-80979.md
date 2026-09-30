# #80979: SearchableSelectControl: Add to @wordpress/ui

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`487a59c`](https://github.com/WordPress/gutenberg/commit/487a59cc66661ea8620db56bfaa41769180fe743)
- **Discussion:** [#80979](https://github.com/WordPress/gutenberg/pull/80979) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a new `SearchableSelectControl` component to the `@wordpress/ui` package, composing the existing `Field` and `SearchableSelect` primitives into a complete form control with integrated label, description, and details. The component forwards the trigger ref to the underlying `SearchableSelect` button and exposes `Group`, `GroupLabel`, `Item`, `Collection`, and `useFilteredItems` as attached sub-components. It is marked `use-with-caution` in Storybook, pending review of style consistency with `@wordpress/components`.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** A new form control is available for searchable single-selection fields that need standard label/description patterns. Import via `import { SearchableSelectControl } from '@wordpress/ui'`.
- **No action required** for existing code. This is purely additive; no existing exports, props, or behavior change.
- The `SearchableSelect` primitive's JSDoc now includes a note: "Prefer `SearchableSelectControl` when using with a standard label and description." This is documentation-only, not a deprecation.
- The component is explicitly **not yet recommended** for production use alongside `@wordpress/components` (see issue #76135). Treat it as a design-system preview.

## Technical details

The new component lives at `packages/ui/src/form/searchable-select-control/searchable-select-control.tsx`. It is a `forwardRef<HTMLButtonElement, SearchableSelectControlProps>` that renders:

```tsx
<Field.Root className={ className }>
  <Field.Label hideFromVision={ hideLabelFromVision }>
    { label }
  </Field.Label>
  <SearchableSelect ref={ ref } aria-label={ label } { ...restProps } />
  { description && <Field.Description>{ description }</Field.Description> }
  { details && <Field.Details>{ details }</Field.Details> }
</Field.Root>
```

The barrel export in `packages/ui/src/form/searchable-select-control/index.ts` attaches sub-components via `Object.assign`:

```ts
export const SearchableSelectControl = Object.assign(
  _SearchableSelectControl,
  { Group, GroupLabel, Item, Collection, useFilteredItems }
);
```

These are re-exported from the existing primitives (`../primitives/combobox/group`, `../primitives/combobox/group-label`, `../primitives/searchable-select/item`, `../primitives/combobox/collection`, `../primitives/combobox/use-filtered-items`).

The component is exported from the package's form index (`packages/ui/src/form/index.ts`) alongside the other `*Control` components.

Tests (`test/index.jsdom.test.tsx`) verify: ref forwarding to the combobox trigger, accessible label/description via `getByRole('combobox', { name, description })`, popup dialog naming, and `FormData` submission with both custom `name` and `defaultValue`.

## Contribution

Opened by @mirka with co-authorship from @ciampo. The only substantive discussion was @mirka asking whether cross-references should be added to `@wordpress/components` Storybook ("use X component instead" recommendations); the answer was to defer that to a separate pass once the component set is marked ready. An unlinked contributor, @cursoragent, is credited in the merge commit, suggesting AI-assisted authorship.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
