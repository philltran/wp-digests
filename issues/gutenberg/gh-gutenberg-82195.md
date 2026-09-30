# #82195: UI: Add Combobox and Autocomplete Status

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`a877f67`](https://github.com/WordPress/gutenberg/commit/a877f674d244ba1cb12c3a5d727f3efd4c4d0256)
- **Discussion:** [#82195](https://github.com/WordPress/gutenberg/pull/82195) · 7 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Adds `Combobox.Status` and `Autocomplete.Status` subcomponents to `@wordpress/ui`. They are thin wrappers around the Base UI `Status` primitive and give async item popups a polite live region (`role="status"`, `aria-live="polite"`, `aria-atomic="true"`) for announcing loading or result-count messages. Before this, the components had no declarative way to announce async list state. Higher-level components such as `SearchableChipSelect` and `SelectControl` do not get `Status` in this PR.

## Impact

**Plugin & theme developers using `@wordpress/ui`**
- New opt-in subcomponents: `Autocomplete.Status` and `Combobox.Status`. Existing code is unaffected, so no action is required.
- If you build async search or loading popups on these primitives, you can render `Status` inside the popup to announce loading and result counts.
- The component docs say to keep `Status` mounted and change or omit its children instead. Don't hide it with `display: none`, `hidden` or `aria-hidden`, and don't conditionally omit the component, or announcements can be lost.

**Higher-level components**
- `SearchableChipSelect` and `SelectControl` are not wired up to `Status`. That is left to a follow-up.

**Announcement guidance**
- In discussion, the author recommended consolidating on the declarative `Status` rather than mixing it with imperative `speak()`. This is a stated direction, not an enforced rule.

## Technical details

**New files** (each is a `forwardRef` wrapper over `@base-ui/react/{autocomplete,combobox}` `Status`):
- `packages/ui/src/form/primitives/autocomplete/status.tsx`
- `packages/ui/src/form/primitives/combobox/status.tsx`

Each wrapper merges `itemPopupStyles.status` (from `utils/css/item-popup.module.css`) into `className` via `clsx`, and forwards `ref` and the remaining props. It is exported as `Status` from each primitive's `index.ts`. `AutocompleteStatusProps` and `ComboboxStatusProps` are added to the `types.ts` files as `ComponentProps<typeof _X.Status>` plus `children`.

The CHANGELOG entry says item popups give `Status` its own collapsing grid row, so a visible result count sits above the list without overlaying items. The CSS for this is in `item-popup.module.css`, which falls in the truncated part of the diff and can't be verified here.

Usage pattern from the updated stories:

```tsx
function AsyncStatus( { loading } ) {
  const filtered = BaseCombobox.useFilteredItems();
  return (
    <Combobox.Status>
      { loading ? <><Spinner /> Loading…</> : <VisuallyHidden>{ filtered.length } results found.</VisuallyHidden> }
    </Combobox.Status>
  );
}
// Inside the popup: <AsyncStatus/> then <Combobox.Empty>{ loading ? null : 'No results found.' }</Combobox.Empty>
```

**Stories:** The `AsyncItems` stories for both components are rewritten to use `Status` with a spinner and "Loading…", replacing the old "Loading..." text or disabled loading item. They also clear pending timeouts via a `useRef`. New `AsyncItemsVisibleCount` stories show the count visibly. The Combobox story now loads on popup open rather than on mount.

**Tests:** The jsdom ref-forwarding tests add a `statusRef` and assert `role="status"`, `aria-live="polite"` and `aria-atomic="true"` on it (and on `Empty`). The Base UI keep-mounted live-region behavior is not retested.

No hooks, REST, or DB changes. The bundle-size bot reports a small increase of about +59 to +139 B in the block-editor bundle.

## Contribution

Opened by @mirka, with @Mamaduka and @jasmussen commenting. @Mamaduka asked how consumers should announce async filter/search results, since `FormTokenField` did it via `updateSuggestions` and #80967 could use `debouncedSearch` + `speak`. @mirka responded by adding Storybook examples (visually hidden versus visible count) and argued for standardizing on declarative `Status` over imperative `speak()`, noting the concern about too many live regions should be measured before optimizing. @jasmussen found the documented async pattern fine and said he favors optimistic UI that shows fallbacks only after a delay.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
