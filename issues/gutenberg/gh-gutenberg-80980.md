# #80980: SearchableChipSelectControl: Add form control to @wordpress/ui

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`2fa5b8c`](https://github.com/WordPress/gutenberg/commit/2fa5b8ce684f1b878b93a4cacc7f642f6d0dc4f9)
- **Discussion:** [#80980](https://github.com/WordPress/gutenberg/pull/80980) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `SearchableChipSelectControl` to `@wordpress/ui`: a higher-level form control that composes `Field` (label, description, details) around the `SearchableChipSelect` primitive, following the pattern of `SelectControl` and `InputControl`. As part of the change, `SearchableChipSelect` now forwards its `ref` to the search input rather than the chips container, and both components are marked `recommended` in Storybook and in the ESLint plugin's `use-recommended-components` rule.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:**
  - A new `SearchableChipSelectControl` is available with `label`, `description`, `details`, and `hideLabelFromVision` props, plus `Group`, `GroupLabel`, `Item`, `ChipWithRemove`, and `Collection` subcomponents.
  - **Behavior change:** the `ref` on `SearchableChipSelect` now resolves to an `HTMLInputElement` (the search input) instead of an `HTMLDivElement` (the chips container). Any consumer that relied on the old ref type, for example to measure or position the container, must update. The CHANGELOG records this under a fix entry rather than as breaking.
- **ESLint plugin users:** `@wordpress/use-recommended-components` now allowlists `SearchableChipSelect` and `SearchableChipSelectControl` from `@wordpress/ui`.
- **Site owners / hosting / REST consumers:** no impact.

## Technical details

**New component** (`packages/ui/src/form/searchable-chip-select-control/`):

```tsx
<Field.Root className={ className }>
	<Field.Label hideFromVision={ hideLabelFromVision }>{ label }</Field.Label>
	<SearchableChipSelect ref={ ref } { ...restProps } />
	{ description && <Field.Description>…</Field.Description> }
	{ details && <Field.Details>…</Field.Details> }
</Field.Root>
```

The component is a `forwardRef<HTMLInputElement, SearchableChipSelectControlProps>`. `index.ts` attaches `Group`, `GroupLabel`, `Item`, `ChipWithRemove`, and `Collection` (imported from `../primitives/combobox/*`) via `Object.assign`. It is exported from `packages/ui/src/form/index.ts` as a named export.

**Ref change** in `searchable-chip-select.tsx`: the generic type goes from `HTMLDivElement` to `HTMLInputElement`. The `ref` is removed from the chips container element and passed to `Combobox.Input`. The unit test now asserts that `ref.current` is the `combobox` role element and that it can receive focus.

**ESLint:** `rules/use-recommended-components.js` adds both names to the `@wordpress/ui` allowlist.

**Storybook:**
- The primitive's status changes from `use-with-caution` to `recommended`, and its `manifest` tag is added.
- The `Creatable` and `GroupedCreatable` stories move out of the primitive's stories. The visible `Creatable` story is now in the Control's stories, with a grouped variant likely alongside it (the diff is truncated).
- Both roots now set an explicit `render` as a workaround for the upstream Storybook issue storybookjs/storybook#34877, which affects the components manifest extractor.
- The Control's stories reuse the primitive's stories and add `VisuallyHiddenLabel`, `WithDetails`, `WithCustomSearchPlaceholder`, and `WithDisabledOption`.

JSDoc on the primitive now directs users to prefer the Control when a standard label and description are needed. The diff was truncated, so the Control's `types.ts` and any tests for it were not visible.

## Contribution

Follow-up to the `SearchableChipSelect` primitive in #80779. In review, @georgestephanis asked whether chips can be reordered by dragging. @mirka pointed to the long-standing issue #22048 and said the core design system team won't build reordering unless a core use case appears. She said the modular primitive architecture is meant to let consumers build such behavior, and that gaps in the primitives (such as `Combobox`) would be addressed if they block it.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
