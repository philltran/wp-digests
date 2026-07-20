# #80208: Widget Dashboard: collapse inline attribute controls into a dropdown

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Widget Dashboard`
- **Merged:** [`284e02b`](https://github.com/WordPress/gutenberg/commit/284e02b718c6355f9bbc5167fa5feecbc806829c)
- **Discussion:** [#80208](https://github.com/WordPress/gutenberg/pull/80208) · 7 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/widget-dashboard` package now dynamically collapses inline `relevance: 'high'` attribute controls into a dropdown when the widget tile header lacks horizontal space. Instead of overflowing or clipping, the promoted fields unmount and render inside a `Popover` triggered by a `settings` icon, while the dedicated settings trigger (now using the `drawerRight` icon) remains visible in the toolbar. This prevents layout breakage on narrow dashboards without requiring host developers to manually manage tile widths.

## Impact

- **Host developers & plugin/theme authors:** No immediate action required. The change is internal to the `@wordpress/widget-dashboard` package and automatically applies to widgets using the `relevance` attribute declaration.
- **Custom dashboard builders:** If you manually override `WidgetAttributeControls` or rely on the previous inline-only layout, you will need to account for the new collapsed dropdown presentation. The settings trigger icon changed from `moreVertical` to `drawerRight`.
- **Breaking changes:** None. The `relevance` attribute remains a hint, and the host retains placement control. No migration or configuration changes are required.

## Technical details

The diff introduces three new files in `packages/widget-dashboard/src/components/widget-attribute-controls/`:
- `use-inline-controls-fit.ts`: A custom hook using `useResizeObserver` to measure the inline controls' natural width against the header's available budget (via `useWidgetHeaderAvailableSize`). It accepts `reservedSize` and `locked` options, returns a `collapsed` boolean and a `measureRef`, and uses a `held` state to prevent layout shifts while the user interacts with either surface.
- `attribute-controls-dropdown.tsx`: Renders the collapsed state using `@wordpress/ui`'s `Popover.Root`, `Popover.Trigger`, and `Popover.Popup`, wrapping a `DataForm` from `@wordpress/dataviews`.
- `widget-attribute-controls.tsx`: Refactored to conditionally mount either the inline `DataForm` (wrapped in a `Stack` with `measureRef`) or the `AttributeControlsDropdown`. It calculates a `triggerReserve` size to exclude the settings trigger from the collapse budget and tracks focus states (`inlineHasFocus`, `dropdownTriggerHasFocus`) to lock the presentation during interaction.

Before/after pattern for the toolbar layout:
```tsx
// Before: Always inline
<DataForm data={ data } fields={ fields } form={ form } onChange={ handleChange } />

// After: Conditional mount based on fit measurement
{ ! collapsed ? (
    <Stack ref={ measureRef } className={ styles[ 'inline-controls' ] }>
        <DataForm data={ data } fields={ fields } form={ form } onChange={ handleChange } />
    </Stack>
) : (
    <AttributeControlsDropdown fields={ fields } data={ data } onChange={ handleChange } onOpenChange={ setDropdownOpen } />
)}
```
Documentation in `docs/explanations/architecture/dashboard-widgets.md` and `packages/widget-dashboard/README.md` was updated to reflect the `relevance` mapping to surfaces and the new collapse behavior.

## Contribution

Opened by @retrofox and merged in commit `284e02b`. Co-authored by @chihsuan, @ciampo, and @mirka. The PR addresses a layout overflow issue when multiple `relevance: 'high'` attributes are declared. During review, @mirka raised a concern about using `@wordpress/ui`'s `Popover` with `DataForm` content due to compatibility slot wiring; @ciampo opened PR #80278 to address the compatibility layer first, allowing this PR to proceed. The team also noted future considerations for the settings surface (modal vs. removal) and identity reserve calibration, but these were deferred to follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
