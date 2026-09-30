# #82556: UI: Add CheckboxGroup form primitive

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`4c92bc4`](https://github.com/WordPress/gutenberg/commit/4c92bc42c3e8bf6ded087294aa8662ce769b8269)
- **Discussion:** [#82556](https://github.com/WordPress/gutenberg/pull/82556) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/ui` gains a `CheckboxGroup` form primitive, a thin wrapper over Base UI's `CheckboxGroup` that shares one selected-values state across child checkboxes. It also ships a `CheckboxGroup.NestedItems` layout wrapper for indenting children, and supports tri-state parent checkboxes via `allValues` plus a `parent` prop on `Checkbox`/`CheckboxControl`. A new "Checkbox Groups" Storybook docs page shows how to compose labeled groups with `Fieldset`.

## Impact

- **Plugin/theme developers using `@wordpress/ui`:** New export `CheckboxGroup` (with `CheckboxGroup.NestedItems`) from the form primitives. Purely additive; no existing API changes.
- **Status:** The story metadata marks it `use-with-caution` and not yet recommended alongside `@wordpress/components`, pending style-consistency review (see Gutenberg #76135). The PR states the checkbox family is not yet marked recommended.
- **Site owners / hosting / REST consumers:** Not affected.
- **Action required:** None. Opt in if you build forms with `@wordpress/ui` and need grouped or nested/tri-state checkboxes.

## Technical details

**New files** under `packages/ui/src/form/primitives/checkbox-group/`:

- `checkbox-group.tsx`: wraps `CheckboxGroup` from `@base-ui/react/checkbox-group` with `forwardRef`. The default `render` is `<Stack direction="column" gap="sm" />`; callers can override `render` (e.g. pass `CheckboxGroup` to `Fieldset.Root`'s `render` prop).
- `nested-items.tsx`: `CheckboxGroup.NestedItems`, a `Stack` (column, `gap="sm"`) with a `nested-items` class.
- `style.module.css`: inside `@layer wp-ui` / `components`, sets `padding-inline-start` to `calc(var(--wp-ui-checkbox-input-size, var(--wpds-dimension-size-2xs)) - var(--wpds-dimension-size-5xs))`.
- `index.ts`: composes the export with `Object.assign( _CheckboxGroup, { NestedItems } )` and sets `displayName = 'CheckboxGroup.NestedItems'`.
- `types.ts`: `CheckboxGroupProps` (Base UI props plus `children`) and `CheckboxGroupNestedItemsProps` (`Stack` props).

`form/primitives/index.ts` now exports `CheckboxGroup`. JSDoc on `Checkbox` and `CheckboxControl` links to the new docs page.

Usage for a parent checkbox (controlled mode):

```tsx
<CheckboxGroup
  value={ value }
  onValueChange={ setValue }
  allValues={ [ 'apple', 'orange', 'banana' ] }
  aria-label="Fruit"
>
  <CheckboxControl parent value="fruit" label="Fruit" />
  <CheckboxGroup.NestedItems>
    <CheckboxControl value="apple" label="Apple" />
    ...
  </CheckboxGroup.NestedItems>
</CheckboxGroup>
```

JSDoc and stories recommend not nesting more than one level for screen reader accessibility, and giving each `CheckboxGroup` its own accessible name when a `Fieldset` contains several. jsdom tests cover ref forwarding, multi-select in uncontrolled mode, parent select-all via `allValues`, and the mixed state (`toBePartiallyChecked`). Also adds a CHANGELOG entry, a `CheckboxGroup` story, and the `checkbox-groups.mdx`/`.story.tsx` docs page.

## Contribution

Authored by @mirka as part of the broader `@wordpress/ui` work tracked in #74178, with @ciampo credited in the props list. The description notes that spacing around the fieldset description is slightly off and is deferred to #82729, and that marking the checkbox family as recommended will be a separate change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
