# #79348: DataForm panel layout: fix double-clicking a field row leaving the flyout stuck open

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`e5154d2`](https://github.com/WordPress/gutenberg/commit/e5154d2d280aa252fa3a6a3985924381132f4527)
- **Discussion:** [#79348](https://github.com/WordPress/gutenberg/pull/79348) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the `DataForm` panel layout where double-clicking a field row or dragging to select text would leave the edit flyout stuck open or block subsequent flyouts from opening. The fix updates the row click handler to skip the text-selection guard for double-clicks, already-open flyouts, and the edit button, ensuring predictable dialog behavior in the Site Editor quick edit flow.

## Impact

- **Site Editor & `DataViews` consumers:** Resolves a UI bug where the quick-edit flyout could become unresponsive after double-clicking or selecting text in a field row.
- **Plugin & theme developers:** No action required for standard usage. If you are directly extending or overriding `packages/dataviews/src/components/dataform-layouts/panel/summary-button.tsx`, note that the `SummaryButton` component now accepts `isOpen` instead of `aria-expanded`.

## Technical details

The fix lives in `packages/dataviews/src/components/dataform-layouts/panel/summary-button.tsx`. The `handleRowClick` handler now accepts a `React.MouseEvent` and evaluates four conditions before calling `onClick()`:

- `!isOpen` (skip if flyout is already open)
- `event.detail < 2` (skip double-clicks)
- `!editButtonRef.current?.contains( event.target )` (skip clicks on the edit pencil button)
- `rowRef.current?.ownerDocument.defaultView?.getSelection()?.toString()` (skip if text is selected)

If any condition is false, the handler proceeds to open/close the flyout. The component signature was updated to accept `isOpen: boolean` instead of `'aria-expanded'?: boolean`, and this prop is now threaded through `modal.tsx` and `dropdown.tsx` to `SummaryButton` and the edit `<Button>`.

**Before:**
```tsx
const handleRowClick = () => {
	const selection = rowRef.current?.ownerDocument.defaultView?.getSelection();
	if ( selection && selection.toString().length > 0 ) return;
	onClick();
};
```

**After:**
```tsx
const handleRowClick = ( event: React.MouseEvent ) => {
	if ( !isOpen && event.detail < 2 && !editButtonRef.current?.contains( event.target ) && rowRef.current?.ownerDocument.defaultView?.getSelection()?.toString() ) {
		return;
	}
	onClick();
};
```

## Contribution

The issue was flagged by @jasmussen during review of #76934. @ntsekouras opened #79348 to address the guard logic, but @jasmussen caught a TypeScript compilation error when `isOpen` was added to `SummaryButton` without being passed down. After a quick fix, @ciampo noted a related structural rework in #79275; @ntsekouras filed #79427 to track it separately and proposed landing this PR as an interim fix, which @ciampo approved. The change was merged shortly after.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
