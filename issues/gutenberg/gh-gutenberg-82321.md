# #82321: Editor: Refactor the View menu to use the Menu component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Tool] E2E Test Utils`
- **Merged:** [`df85697`](https://github.com/WordPress/gutenberg/commit/df85697e927aabfab04fe4f6d21f3c99d1ada590)
- **Discussion:** [#82321](https://github.com/WordPress/gutenberg/pull/82321) · 16 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The editor header's View menu (the device preview dropdown) is rebuilt on the `Menu` component from `@wordpress/ui`, replacing `DropdownMenu`, `MenuGroup`, `MenuItem`, and `MenuItemsChoice` from `@wordpress/components`. Viewport choices are now a radio group, "Responsive styles" and "Show template" are checkbox items, and "View site" and "Preview" are link items. This brings it in line with the Options menu, which was already migrated. The preview menu item's accessible name changes from "Preview in new tab" to "Preview (opens in a new tab)".

## Impact

**Plugin developers**
- Items registered in the `core/plugin-preview-menu` slot (via `PluginPreviewMenuItem`) are now rendered through the `MoreMenuItem`/`MoreMenuGroup` adapters used by the Options menu. The `as` prop is ignored (deprecated in #82319). The CHANGELOG adds a deprecation note that link items honor only `target="_blank"`.
- The slot's `fillProps` no longer passes `onClick: onClose`; the fills are wrapped in a headingless group instead.

**E2E / test authors**
- Any selector matching the `menuitem` named "Preview in new tab" must change to "Preview (opens in a new tab)". The Playwright utility `editor.openPreviewPage()` and the in-repo specs were updated accordingly.
- Viewport options are now `menuitemradio` items and the toggles are `menuitemcheckbox` items in the `Menu` implementation, rather than the previous `MenuItemsChoice` markup. Tests relying on the old roles or DOM structure may need review.

**Theme/plugin CSS**
- The `.editor-preview-dropdown` wrapper class no longer exists. The toggle is now `.editor-preview-dropdown__toggle` (with `is-responsive-editing` on the button itself), and the popup has `.editor-preview-dropdown__popup`. Custom CSS targeting the old wrapper will stop matching.

**Site owners**
- No action required. Keyboard behavior and styling now match the Options menu.

## Technical details

**`preview-dropdown/index.jsx`**
- The component is split: the default export `PreviewDropdown` only calls `useViewportMatch( 'medium', '<' )` and returns `null` on small screens, otherwise rendering the new inner `PreviewMenu`. This keeps store subscriptions from running where the menu doesn't render.
- `PreviewMenu` renders `Menu.Root` (`modal={ false }`, `disabled`) with a `Menu.Trigger` whose `render` prop is a `@wordpress/components` `Button` (compact, device icon, `label="View"`, `showTooltip`, `accessibleWhenDisabled`). The popup is `Menu.Popup` with `positioner={ <Menu.Positioner align="end" /> }`.
- Viewport choices use `Menu.RadioGroup` (`value`/`onValueChange`) with `Menu.RadioItem`, `Menu.ItemLabel`, and `Menu.ItemDescription`. The per-choice `icon` entries were dropped.
- `handleResponsiveEditingChange` now receives the new boolean from `Menu.CheckboxItem`'s `onCheckedChange` instead of inverting local state. "Show template" likewise derives `newRenderingMode` from `checked`.
- "View site" uses `Menu.LinkItem` with `openInNewTab` and `closeOnClick`. The manual `VisuallyHidden` "(opens in a new tab)" text and `external` icon were removed.
- Plugin items: `<ActionItem.Slot name="core/plugin-preview-menu" fillProps={ { as: MoreMenuItem } }>` with children rendering `<MoreMenuGroup>`.

**`more-menu/more-menu-group.tsx`**: `label` becomes optional, and `Menu.GroupLabel` renders only when provided.

**`post-preview-button/index.jsx`**: `PostPreviewMenuItem` now renders `Menu.LinkItem` (`href`, `onClick`, `openInNewTab`, `closeOnClick`, `target`) with a `Menu.ItemLabel` of "Preview". When `previewProps.disabled` is true it renders a disabled `Menu.Item` with no `href`. The `onPreview` prop is no longer passed by the View menu.

**Styles**
- `preview-dropdown/style.scss` adds `.editor-preview-dropdown__popup { max-width: var(--wpds-dimension-surface-width-xs) }` and re-scopes the toggle selectors. `header/style.scss` distraction-free hiding now targets `.editor-header__settings > .editor-preview-dropdown__toggle`.
- `more-menu/style.scss` replaces `--wp-ui-menu-max-width` with a direct `max-width` on the popup.

**Tests:** `post-preview-button` jsdom tests wrap the item in `Menu.Root`/`Menu.Popup`, and the e2e specs and `openPreviewPage` use `getByRole( 'menuitem', { name: 'Preview (opens in a new tab)' } )`.

```diff
- page.click( 'role=menuitem[name="Preview in new tab"i]' )
+ page.getByRole( 'menuitem', { name: 'Preview (opens in a new tab)' } ).click()
```

## Contribution

This is a follow-up to #81564, which migrated the Options menu, and it reuses that PR's adapters. The PR was prepared with Claude assistance. Review discussion centered on the external-link indicator: jasmussen suggested an icon rather than the in-text ↗, but the styling comes from the shared `Menu` link item and matches the `Link` component, so it stayed, with an icon variant possible in a separate PR. A remaining question was whether to align the trigger button label and menu label, and ciampo voted for "View options"; the diff shown still keeps `label="View"` on the trigger.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
