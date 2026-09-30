# #79560: Menu: Add UI component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`eca76ad`](https://github.com/WordPress/gutenberg/commit/eca76ad7688a90f75472ca03ce6fffb94da3bc32)
- **Discussion:** [#79560](https://github.com/WordPress/gutenberg/pull/79560) · 34 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds a `Menu` component to `@wordpress/ui`, built on Base UI's menu primitive. It supports regular, link, checkbox, radio, grouped, and submenu items, with optional `prefix`, `suffix`, and `shortcut` slots plus `Menu.ItemLabel` / `Menu.ItemDescription` subcomponents. It follows the package's overlay, positioning, motion, direction, and keyboard-shortcut conventions and uses WPDS tokens. It is exported as a namespace from the package root (`export * as Menu from './menu'`).

## Impact

**Plugin & theme developers**
- New public API in `@wordpress/ui`: `Menu.Root`, `Trigger`, `Popup`, `Portal`, `Positioner`, `Item`, `LinkItem`, `CheckboxItem`, `RadioGroup`, `RadioItem`, `Group`, `GroupLabel`, `Separator`, `SubmenuRoot`, `SubmenuTrigger`, `ItemLabel`, `ItemDescription`.
- Every item must have `Menu.ItemLabel` as its first direct child, optionally followed by a single `Menu.ItemDescription`. In non-production builds anything else throws an `Error`.
- No migration is required. The legacy ariakit-based `Menu` in `@wordpress/components` is untouched. The PR notes that a compatibility and migration strategy for legacy menu items and extensibility slots (for example the block toolbar "..." menu slot) is still undefined, and is tracked partially in #61094.

**Site owners:** no action required.

**Known gaps (listed as follow-ups in the PR):** dark-palette elevation contrast (#75266), consistent focus/hover indicators across composite components (#80405), and support for multiple `Menu.ItemDescription` components per item.

## Technical details

The component lives under `packages/ui/src/menu/`, with one file per part, each wrapping the corresponding `@base-ui/react/menu` part via `forwardRef` and merging `className` with CSS-module styles.

- **Items:** `Item` and `CheckboxItem` (the diff shows both) share `useItemContent()` and `ItemContent` from `item.tsx`. `ItemContent` renders `prefix`, `suffix`, and `shortcut` (via `KeyboardShortcutDisplay`) plus an optional internal `trailing` slot. Content is placed first in the DOM so Base UI's typeahead, which falls back to `textContent`, works. A CSS grid places the prefix visually before it. The prefix is `aria-hidden`.
- **Accessible naming:** `useItemContent` generates label and description ids with `useId` and wires `aria-labelledby` and `aria-describedby`. A generated `aria-labelledby` is only supplied when the consumer passes neither `aria-label` nor `aria-labelledby`. `useKeyboardShortcutProps` adds `aria-keyshortcuts` and a shortcut description via `KeyboardShortcutDescription`.
- **Context:** `context.tsx` defines `MenuContext` (`isSubmenu`) and `MenuItemContentContext` (`labelId`, `descriptionId`, `labelTrailing`), consumed by `ItemLabel` and `ItemDescription` through `useRender` and `mergeProps`.
- **Checkbox indicator:** `CheckboxItem` renders `_Menu.CheckboxItemIndicator` with `keepMounted` and the `check` icon from `@wordpress/icons`, so the dedicated selection column is reserved. Per the discussion, radio items use the same tick icon and can also take a prefix.
- **Other changes:**
  - `alert-dialog` story `MenuTrigger` is rewritten to use the new `Menu` with a controlled `AlertDialog.Root`, opened via `onClick`, instead of raw Base UI `Menu` primitives.
  - The comment on `ITEM_POPUP_POSITIONER_PROPS` now mentions Autocomplete.
  - `CHANGELOG.md` gains "Add a `Menu` component" and "`Menu`: Add shortcut display and accessibility metadata support".

The diff is truncated, so the popup, positioner, root, trigger, submenu, and CSS module files are not shown here. Storybook stories and a usage-guidelines doc (including guidance on Menu vs. Select) are described in the PR but not visible in the diff.

```jsx
<Menu.Root>
  <Menu.Trigger>Actions</Menu.Trigger>
  <Menu.Popup>
    <Menu.Item onClick={ onEdit }>
      <Menu.ItemLabel>Edit</Menu.ItemLabel>
      <Menu.ItemDescription>Open the editor</Menu.ItemDescription>
    </Menu.Item>
  </Menu.Popup>
</Menu.Root>
```

## Contribution

The author (@ciampo) opened the PR as a port of the layout and features of the ariakit-based `Menu` in `@wordpress/components`, and asked early whether to keep custom abstractions (`prefix`/`suffix`, `ItemLabel`/`ItemDescription`, grid layout) or stay closer to raw Base UI. His position was to keep them, to enforce the design-spec layout and typography. After review from @mirka he changed radio and checkbox to share one tick icon, stopped radio/checkbox marks from replacing a prefix, and detached most styles shared with Select because the layout needs differ. Design review from @fcoveram later pushed back on the shared icon and raised dark-mode contrast, focus-ring visibility on keyboard navigation, submenu motion, RTL chevron direction, and small-viewport submenu placement. Several of these are deferred as follow-ups. @Mamaduka offered help and @ciampo suggested pilot refactors (block toolbar menu, editor options menu, list view item menu, DataViews menus) to expose gaps. The PR states it was implemented with Codex assistance and reviewed locally.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
