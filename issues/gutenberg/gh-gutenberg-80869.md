# #80869: UI: Add Combobox.InputGroup primitive

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`d766a84`](https://github.com/WordPress/gutenberg/commit/d766a848b2103c6d1a577e9c73b08d816bc28ef7)
- **Discussion:** [#80869](https://github.com/WordPress/gutenberg/pull/80869) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/ui` package now exports a `Combobox.InputGroup` primitive that wraps Base UI’s combobox input with a container element. This addition allows developers to compose the combobox input with prefix/suffix slots or other UI elements while ensuring the dropdown popup correctly anchors to the full control boundary rather than just the raw `<input>` element. It also applies `role="group"` to the wrapper for improved accessibility semantics.

## Impact

- **Plugin & theme developers:** Can now wrap `Combobox.Input` with `Combobox.InputGroup` when building composite form controls that include prefixes, suffixes, or custom wrappers.
- **No breaking changes or deprecations.** Existing `Combobox` usage remains fully compatible.
- **No migration required.** The component is opt-in; developers only need to adopt it when their combobox layout requires a composite boundary for popup anchoring.

## Technical details

The diff introduces `packages/ui/src/form/primitives/combobox/input-group.tsx`, a thin `forwardRef` wrapper around `@base-ui/react/combobox`'s `InputGroup`. It is exported via `packages/ui/src/form/primitives/combobox/index.ts` and typed as `ComboboxInputGroupProps` in `types.ts`. The wrapper spreads props and passes a `ref` to the underlying `_Combobox.InputGroup`, which renders a `<div>` with `role="group"`. Previously, developers had to manually manage the anchor boundary or accept that the popup would attach to the bare input. Now:

```tsx
// Before (popup anchors to input only)
<Combobox.Input placeholder="Search" />

// After (popup anchors to the full control boundary)
<Combobox.InputGroup>
  <Combobox.Input placeholder="Search" />
</Combobox.InputGroup>
```

The change also updates the Storybook meta and test suite to verify the new subcomponent renders as an `HTMLDivElement` and accepts refs correctly.

## Contribution

Opened by @mirka and co-authored with @simison, the PR landed with minimal review friction. The implementation deliberately mirrors the existing `Autocomplete.InputGroup` pattern to maintain consistency across the UI package. The author noted it was specifically needed to resolve a popover positioning issue in the `SearchableChipSelect` component, though that broader refactor was handled separately. The diff merged cleanly with zero size impact and standard test coverage.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
