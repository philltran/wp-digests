# #82038: UI: Use root theme for portaled overlays

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`5b4ec99`](https://github.com/WordPress/gutenberg/commit/5b4ec9930bcdda4280a1ad6ef95d10569ff8a31a)
- **Discussion:** [#82038](https://github.com/WordPress/gutenberg/pull/82038) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Portaled overlays in `@wordpress/ui` (AlertDialog, Autocomplete, Combobox, Dialog, Drawer, Menu, Popover, Select) no longer wrap their popup in an internal `ThemeProvider`. They now take their theme from where they render in the DOM: the document-root theme with the default portal, or the custom container's ancestry with a custom portal. Previously they re-emitted the trigger's nearest contextual theme, so an overlay opened from a nested-themed region (e.g. a sidebar) picked up that region's theme. Tooltip keeps its provider because it creates its dark surface.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** This is flagged as a **Breaking Change** in the package CHANGELOG. If you relied on a nested `ThemeProvider` around a trigger to theme its overlay, the overlay will now follow the root theme instead. To theme an overlay differently, use a custom portal container inside the desired theme boundary.
- **Wrapper-dependent CSS/tests:** The `ThemeProvider` wrapper no longer sits between the portal and the popup element. Any custom selectors, e.g. `.backdrop ~ * .popup`, that assumed an intermediate wrapper need to target the popup as a direct sibling.
- **Site owners / end users:** Overlays in the Site Editor and post editor should consistently follow the selected admin color scheme / root theme. This also aligns `@wordpress/ui` with existing `@wordpress/components` overlays.
- **Headless/REST/hosting:** No impact.

## Technical details

The diff removes `<ThemeProvider>` from the `popup.tsx` of alert-dialog, dialog, drawer, autocomplete, combobox, select, menu, and popover under `packages/ui/src/`. Each `_X.Popup` from Base UI is now rendered directly (mostly re-indentation plus the removal of the wrapper and its import from `utils/theme-provider`). Tooltip is untouched.

Other changes:

- `dialog/style.module.css`: the modal-border override selector changes from `.backdrop ~ * .popup` to `.backdrop ~ .popup`, since the popup is now a direct sibling of the backdrop rather than nested under the wrapper. This preserves the transparent border for modal dialogs while non-modal dialogs keep their neutral border.
- `drawer/popup.tsx`: the comment about the `display: contents` focus-trap workaround is dropped; the description says that workaround is obsolete since the Base UI fix shipped, and that the focused Drawer tests still pass.
- New Dialog test: renders a nested `ThemeProvider` (`cornerRadius="pronounced"`) under a root `ThemeProvider isRoot cornerRadius="subtle"`, opens the dialog, and asserts `popup.closest('[data-wpds-corner-radius]')` is `document.documentElement`, i.e. the root provider's theme mirrored onto the document.
- `packages/ui/CHANGELOG.md`: adds a "Breaking Changes" entry under Unreleased.

No public props, hooks, or REST changes.

## Contribution

This PR by @ciampo replaces the earlier #81183, taking a simpler approach of removing the per-overlay providers. @mirka is credited for props. The PR was prepared with Codex and reviewed by the author. The discussion is otherwise limited to bot output and a comment from the author showing trunk-vs-PR screenshots of the menu and dialog in the "Example Application" story.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
