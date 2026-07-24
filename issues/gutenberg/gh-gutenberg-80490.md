# #80490: UI: Add Autocomplete.Row primitive

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`ca0f125`](https://github.com/WordPress/gutenberg/commit/ca0f12506dc8352d8f8455f617931213d05a17cb)
- **Discussion:** [#80490](https://github.com/WordPress/gutenberg/pull/80490) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/ui` package now exposes the `Autocomplete.Row` primitive, wrapping Base UI’s underlying component to enable multi-cell grid layouts within autocomplete dropdowns. Previously, the WordPress UI wrapper only exposed single-item rows, limiting grid-mode autocomplete implementations. This addition allows developers to group multiple `Autocomplete.Item` components into structured rows, which is essential for complex picker interfaces like emoji selectors.

## Impact

- **Plugin & theme developers:** No breaking changes. Developers using `@wordpress/ui` autocomplete primitives can now adopt `Autocomplete.Row` to build grid-based dropdowns.
- **Block authors:** Enables more complex UI patterns (e.g., emoji pickers, multi-column suggestion lists) without resorting to custom DOM manipulation or external libraries.
- **No action required** for existing autocomplete implementations; the change is strictly additive.

## Technical details

The diff introduces `packages/ui/src/form/primitives/autocomplete/row.tsx`, a thin `forwardRef` wrapper around `_Autocomplete.Row` from `@base-ui/react/autocomplete`. It exports the component via `packages/ui/src/form/primitives/autocomplete/index.ts` and defines `AutocompleteRowProps` in `types.ts` to align with Base UI’s props while explicitly typing `children`. To use it, developers must enable `grid: true` on `Autocomplete.Root` to activate two-dimensional keyboard navigation and grid rendering. The Storybook fixture (`index.story.tsx`) demonstrates chunking items into rows using a custom `chunkItems` helper and applying CSS grid styles via the `style` prop. Tests verify that `Autocomplete.Row` correctly forwards refs to the underlying `HTMLDivElement`.

## Contribution

Opened by @mirka to address a gap in the `@wordpress/ui` wrapper where Base UI’s grid row primitive was previously inaccessible. The implementation followed a straightforward thin-wrapper pattern to maintain consistency with other autocomplete primitives. Review focused on ensuring proper ref forwarding and TypeScript typing alignment with Base UI, with no alternative approaches or rejected designs noted in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
