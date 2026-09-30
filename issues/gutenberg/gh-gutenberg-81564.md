# #81564: Editor: Refactor the Options menu to use the Menu component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `Needs Dev Note`, `[Package] Edit Post`, `[Package] Interface`, `[Package] Edit Site`
- **Merged:** [`185ed32`](https://github.com/WordPress/gutenberg/commit/185ed32cb34a7c31680e83409ddc9fcaae0f3724)
- **Discussion:** [#81564](https://github.com/WordPress/gutenberg/pull/81564) · 25 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The editor's **Options** (⋮) menu in the post and site editors has moved from `DropdownMenu` / `MenuItem` in `@wordpress/components` to the `Menu` component in `@wordpress/ui`. The public plugin APIs stay the same: `PluginMoreMenuItem` and `PluginSidebarMoreMenuItem` still work. What changes is how their items render, and the `@wordpress/editor` changelog lists this as a **breaking change**. Link items now honor only `target="_blank"`, and any custom `as` component must forward its ref. The editor-mode switcher is now a real radio group, and the menu can now support submenu flyouts.

## Impact

**Plugin developers**
- Items registered with `PluginMoreMenuItem` / `PluginSidebarMoreMenuItem` now render as `@wordpress/ui` `Menu` items, not `@wordpress/components` `MenuItem`. Test any plugin that adds items to this menu.
- **Link items:** of all `target` values, only `target="_blank"` is honored. Other `target` values are no longer respected.
- **Custom `as` components** passed to these items must forward their ref, for example with `forwardRef`. They are wrapped so they still work with keyboard navigation. An `as` that doesn't forward its ref is a breaking case under the changelog.
- Code that relied on `@wordpress/components` `MenuItem` props or markup inside this menu (such as `icon`, `info`, or `components-menu-item__*` DOM structure) should be checked again.

**Theme / admin CSS authors**
- The popover class `more-menu-dropdown__content` is no longer applied, and its z-index rule was removed from `packages/editor/src/components/header/style.scss`. The popup now uses `editor-more-menu__popup`. Selectors aimed at the old class will stop matching.

**Site owners**
- No action needed. The menu should work as before, with small visual differences. The editor-mode choice (Visual/Code) is now shown as a radio group.

**Known open items noted on the PR:** popovers opened from the new Menu don't return focus correctly when they close. It is also undecided whether slots should warn when they get an incompatible `as` prop.

## Technical details

**Menu shell** (`packages/editor/src/components/more-menu/index.jsx`)
- `DropdownMenu` + `dropdownProps` is replaced by `Menu.Root modal={ false }`, a `Menu.Trigger` that renders a compact `Button` (`moreVertical`, label "Options"), and a `Menu.Popup` using `Menu.Positioner align="end"`.
- `MenuGroup label=…` becomes `Menu.Group` + `Menu.GroupLabel`, with `Menu.Separator` between groups.
- The Help item is now `Menu.LinkItem openInNewTab`. The manual `external` icon and the `VisuallyHidden` "(opens in a new tab)" text were removed.
- `ViewMoreMenuGroup.Slot` and `ToolsMoreMenuGroup.Slot` no longer receive `fillProps={ { onClose } }`.

**Why items must come from the editor.** `@wordpress/ui` is bundled into each package instead of being shared as a script. Each package therefore gets its own `Menu` React context, and a menu part rendered from `edit-post` into the editor's menu would throw. As a result:
- `@wordpress/editor` exposes new **private** APIs, `MoreMenuItem` and `MoreMenuPreferenceItem`. `edit-post` and `edit-site` now use these instead of `MenuItem` / `PreferenceToggleMenuItem` from `@wordpress/preferences`. A new internal `MoreMenuGroup` wraps groups of slotted items.
- `ActionItem` (`@wordpress/interface`) now takes the component to render from the `as` passed in the slot's `fillProps`. The Options menu passes its own item:

```jsx
// Before
<ActionItem.Slot
  name="core/plugin-more-menu"
  label={ __( 'Panels' ) }
  fillProps={ { onClick: onClose } }
/>

// After
<ActionItem.Slot name="core/plugin-more-menu" fillProps={ { as: MoreMenuItem } }>
  { ( items ) => <MoreMenuGroup label={ __( 'Panels' ) }>{ items }</MoreMenuGroup> }
</ActionItem.Slot>
```

**Shortcuts.** Preference items now take a shortcut object instead of a display string. For example, in `edit-post`:

```js
const FULLSCREEN_SHORTCUT = {
  ariaKeyShortcut: ariaKeyShortcut.secondary( 'f' ),
  displayShortcut: displayShortcut.secondary( 'f' ),
  label: shortcutAriaLabel.secondary( 'f' ),
};
```

In the editor, a local `utils/keyboard-shortcut` helper, `getKeyboardShortcut( { character, modifier } )`, builds these objects. `ModeSwitcher` now reads `getShortcutKeyCombination( 'core/editor/toggle-mode' )` from the keyboard-shortcuts store instead of `getShortcutRepresentation`.

**Other item changes**
- `ModeSwitcher`: `MenuItemsChoice` is replaced by `Menu.RadioGroup` / `Menu.RadioItem`, and the `info` text moves to `Menu.ItemDescription`.
- `CopyContentMenuItem`: now a `Menu.Item` that still takes the `useCopyToClipboard` ref.
- Welcome Guide (both editors): no longer a `PreferenceToggleMenuItem` checkbox. It is a plain `MoreMenuItem` whose `onClick` calls `preferencesStore.toggle( scope, 'welcomeGuide' | 'welcomeGuideTemplate' )`.
- Revision mode renders the same `Menu.Root` with only `<ModeSwitcher />`.

The diff shown was truncated, so the full implementations of `more-menu-group.tsx`, `MoreMenuItem`, and `MoreMenuPreferenceItem` are not visible here.

## Contribution

The PR builds on #79560 and also includes #81507, which the author said could ship separately. It was written with Claude's help. In review, @jasmussen reported an icon-position issue, which @Mamaduka traced to an existing bug on trunk. He also asked for the Top toolbar / Distraction free options to become radio items, which is waiting on @ciampo's work on `Menu` radios. Because the new `Menu` can do submenu flyouts, @Mamaduka pushed a trial "Help" submenu based on @jasmussen's mockup. @jasmussen then decided against a Help flyout for now: Welcome Guide shares a slot with "Manage patterns" and can't easily be moved into it. A "More actions" grouping was mentioned as a possible later follow-up. The PR has the `Needs Dev Note` label.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
