# #78738: Dashboard: Move layout settings to customize toolbar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`1dae94d`](https://github.com/WordPress/gutenberg/commit/1dae94dbf5ed1baca3084fc2359765bc3adef1a4)
- **Discussion:** [#78738](https://github.com/WordPress/gutenberg/pull/78738) · 3 comments · 0 reactions

## Summary

Layout settings in the Gutenberg widget dashboard have been relocated from the more-actions (⋯) menu into the customize toolbar, sitting alongside the **Add widget** button and rendered only while customize/edit mode is active. Previously the more-menu entry was disabled during edit mode, blocking access at exactly the point it is most relevant. The layout settings drawer is now modal — a backdrop with focus trap is rendered, and dismissing via backdrop or Cancel discards only staged grid-layout changes, leaving any widget repositioning made in the same customize session intact. The drawer also closes automatically when the user exits customize mode.

## Impact

**Site owners / admins using the Gutenberg widget dashboard**
- Layout settings are now discoverable in the toolbar during customize mode rather than buried in the more-menu (where they were disabled during edit).
- No configuration required.

**Plugin / extension developers integrating the widget dashboard**
- `cancel()` in `dashboard-context.tsx` now accepts an optional `{ revertLayout?: boolean }` argument (default `true`). Code that wraps or mocks the dashboard context's cancel callback must accommodate this signature change.
- The layout settings drawer's `modal` and `disablePointerDismissal` props are removed; if your code extends `layout-settings.tsx` and relied on the non-modal, pointer-permissive behavior, that assumption no longer holds.

**No action required** for sites not using the Gutenberg widget dashboard feature. There are no PHP, REST API, `block.json`, or database changes.

## Technical details

Three components under `routes/dashboard/widget-dashboard/` are modified:

**`actions.tsx`**
- The "Layout settings" `DropdownMenuItem` inside the more-actions menu is removed.
- A new "Layout settings" `Button` (layout icon on non-mobile viewports, text-only on mobile) is added directly to the edit toolbar, conditionally rendered when `onGridSettingsChange` is provided.
- A `useEffect` observes `editMode`; when it transitions to `false`, the layout settings drawer is closed.

**`dashboard-context.tsx`**
- `cancel()` is extended with an optional parameter object: `cancel({ revertLayout?: boolean })`, defaulting to `true`.
- When `revertLayout` is `false`, only staged grid settings are reverted; widget position changes accumulated during the same customize session are preserved.

```tsx
// Before — cancel always reverted everything
cancel();

// After — dismiss layout settings drawer without wiping widget moves
cancel({ revertLayout: false });

// Exiting customize entirely — still reverts everything (default unchanged)
cancel();
```

**`layout-settings.tsx`**
- `modal={ false }` and `disablePointerDismissal` are removed from the `<Drawer>` call, restoring default modal behavior: a backdrop is rendered, pointer events outside the drawer are intercepted, and focus is trapped inside the panel.

**Tests**
- `routes/dashboard/widget-dashboard/test/actions.test.tsx`: asserts the Layout settings button appears in the toolbar and is absent from the more-menu.
- `routes/dashboard/widget-dashboard/test/staging.test.tsx`: asserts that `cancel({ revertLayout: false })` reverts grid settings only, leaving widget layout staging intact.

## Contribution

Authored and opened by @jameskoster. Merged at commit `1dae94d`. The only discussion on the PR was the automated bundle-size bot confirming a net-zero size change (8.21 MB total, 0 B delta). No alternative approaches or design debate are documented in the available discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
