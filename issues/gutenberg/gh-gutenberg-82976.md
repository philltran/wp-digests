# #82976: Editor: Gather the View group of the Options menu into submenus

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Editor`
- **Merged:** [`cf5d0b0`](https://github.com/WordPress/gutenberg/commit/cf5d0b0e6dc9873b73f53e205cadb230764c30f9)
- **Discussion:** [#82976](https://github.com/WordPress/gutenberg/pull/82976) · 18 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The View group in the editor's Options (ellipsis) menu is restructured from flat items into two submenus: **Appearance** (containing Top toolbar, Distraction free, Spotlight mode, and the `ViewMoreMenuGroup` slot fills such as Fullscreen mode) and **Panels** (containing sidebars registered via `PluginSidebar` and `PluginMoreMenuItem`). The change follows a design proposed by @jasmussen and replaces the former flat `MoreMenuGroup` rendering with a new `MoreMenuSubmenu` component built on `@wordpress/ui`'s `Menu.SubmenuRoot` / `Menu.SubmenuTrigger` / `Menu.Popup` primitives.

## Impact

- **Plugin developers:** Sidebars registered through `PluginSidebar` and `PluginMoreMenuItem` now appear one level deeper in the Options menu, under a "Panels" submenu rather than as top-level items in the menu. The registration API is unchanged — no code modifications are required. Pinned toolbar buttons remain the primary quick-access route.
- **Plugin & theme developers:** The internal `MoreMenuGroup` component (`packages/editor/src/components/more-menu/more-menu-group.tsx`) is removed and replaced by `MoreMenuSubmenu`. This is not a public export of `@wordpress/editor`, so no external code should be importing it. The `toMenuItems` helper is now exported from the module for internal reuse.
- **Site owners / editors:** The Options menu is slightly more nested; toggling Top toolbar, Distraction free, Spotlight mode, or Fullscreen mode now requires opening the Appearance submenu first. Keyboard navigation (ArrowRight to open, ArrowLeft to close) is supported.
- **No action required** for plugin or theme code. No public API, hook, or REST route changes.

## Technical details

The core change is in `packages/editor/src/components/more-menu/index.jsx`. The View `Menu.Group` previously rendered `MoreMenuPreferenceItem` elements (for `fixedToolbar`, `distractionFree`, `focusMode`) and the `ViewMoreMenuGroup.Slot` as direct children. The `core/plugin-more-menu` `ActionItem.Slot` was a separate group below the `ModeSwitcher`. Now both are wrapped in `MoreMenuSubmenu` components inside the View group:

```jsx
// Before
<Menu.Group>
  <Menu.GroupLabel>View</Menu.GroupLabel>
  <MoreMenuPreferenceItem scope="core" name="fixedToolbar" … />
  <MoreMenuPreferenceItem scope="core" name="distractionFree" … />
  <MoreMenuPreferenceItem scope="core" name="focusMode" … />
  <ViewMoreMenuGroup.Slot />
</Menu.Group>
<Menu.Separator />
<ModeSwitcher />
<ActionItem.Slot name="core/plugin-more-menu" fillProps={{ as: MoreMenuItem }}>
  { (items) => <MoreMenuGroup label="Panels">{ items }</MoreMenuGroup> }
</ActionItem.Slot>

// After
<Menu.Group>
  <Menu.GroupLabel>View</Menu.GroupLabel>
  <MoreMenuSubmenu label="Appearance">
    <MoreMenuPreferenceItem scope="core" name="fixedToolbar" … />
    <MoreMenuPreferenceItem scope="core" name="distractionFree" … />
    <MoreMenuPreferenceItem scope="core" name="focusMode" … />
    <ViewMoreMenuGroup.Slot />
  </MoreMenuSubmenu>
  <ActionItem.Slot name="core/plugin-more-menu" fillProps={{ as: MoreMenuItem }}>
    { (items) => <MoreMenuSubmenu label="Panels">{ toMenuItems(items) }</MoreMenuSubmenu> }
  </ActionItem.Slot>
</Menu.Group>
<Menu.Separator />
<ModeSwitcher />
```

The file `more-menu-group.tsx` is renamed to `more-menu-submenu.tsx`. The old `MoreMenuGroup` (which rendered a `Menu.Separator` + `Menu.Group` + optional `Menu.GroupLabel`) is replaced by `MoreMenuSubmenu`, which renders:

```jsx
<Menu.SubmenuRoot>
  <Menu.SubmenuTrigger>
    <Menu.ItemLabel>{ label }</Menu.ItemLabel>
  </Menu.SubmenuTrigger>
  <Menu.Popup>{ children }</Menu.Popup>
</Menu.SubmenuRoot>
```

The `toMenuItems` function (previously module-private) is now exported. It adapts legacy `MenuItem`-based fills (those without an `href` prop) into `MoreMenuItem` wrappers so keyboard navigation works inside the submenu.

In `packages/editor/src/components/preview-dropdown/index.jsx`, the preview dropdown's use of `MoreMenuGroup` is replaced with an inline `Menu.Separator` + `Menu.Group` + `toMenuItems(items)` since the preview menu does not use the submenu pattern.

In `style.scss`, `max-width` is changed to `width` on the more-menu container to give the submenu trigger a fixed width.

E2E tests in `test/e2e/specs/editor/plugins/plugins-api.spec.js` gain an `openPanelsMenu` helper that clicks the Options button then the "Panels" menuitem. `dropdown-menu.spec.js` adds a step verifying ArrowRight opens the Appearance submenu and ArrowLeft closes it. `fullscreen-mode.spec.js` is updated to navigate through the Appearance submenu before clicking Fullscreen mode.

## Contribution

Opened by @adamsilverstein, implementing a design @jasmussen sketched in a comment on PR #76024. The PR initially sat on the `try/notes-in-canvas-margin` branch as part of a three-PR dependency chain (#79864 → #76024 → #82976). @t-hamano flagged that the submenu restructuring was independent of the Notes work and requested a rebase to trunk, which @adamsilverstein did. @Mamaduka raised a design concern that top-level menu items acting as submenu triggers was an unfamiliar pattern and noted an icon-consistency question (mixed icon/no-icon items in the more menu); @jasmussen approved the approach. The code and PR description were generated with Claude Code, with @adamsilverstein reviewing and testing. Co-authored with Mamaduka, t-hamano, jasmussen, and ntsekouras.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
