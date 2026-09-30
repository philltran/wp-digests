# #82087: UI: Add popupWidth prop to item popup components

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Block editor`, `[Package] UI`
- **Merged:** [`9782cfa`](https://github.com/WordPress/gutenberg/commit/9782cfaf13010095b68840cfeec501018b97756f)
- **Discussion:** [#82087](https://github.com/WordPress/gutenberg/pull/82087) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`@wordpress/ui` item popup components gain a `popupWidth` prop with presets `anchor` (default), `content`, `sm`, `md`, `lg`, and `available`. It is wired through `Select.Popup`, `Combobox.Popup`, `Autocomplete.Popup`, and the derived `SelectControl`, `SearchableChipSelect`, and `SearchableChipSelectControl`. It fixes popups growing wider than their trigger when labels contain long unbreakable words or when creatable footer content changes. Because the primitives now default to anchor width, the PR is labeled a breaking change.

## Impact

**Plugin & theme developers using `@wordpress/ui`**
- **Breaking default:** `Autocomplete.Popup`, `Combobox.Popup`, `Select.Popup`, `SearchableChipSelect`, and `SearchableChipSelectControl` now default to a fixed anchor width. Pass `popupWidth="content"` to restore content-sized width (between the anchor width and available viewport bounds).
- `SelectControl` defaults to `content`, so its popup sizing is preserved. The CHANGELOG states this, and the PR description says it matches native `select` behavior.
- Long labels now wrap inside fixed-width popups, because popup items get `overflow-wrap: anywhere`.
- Custom CSS that targeted popup width or relied on popups growing past the anchor should be re-checked.

**Block editor**
- The Position control in `packages/block-editor/src/hooks/position.jsx` now passes `popupWidth="anchor"`. The bespoke `.block-editor-hooks__position-control-item` max-width rule was removed. No action required.

**Site owners / headless / hosting:** No action required.

`Menu` was not changed; the discussion left the question of adding presets there open.

## Technical details

A shared `ItemPopupWidth` type and `ItemPopupWidthProps` are added in `packages/ui/src/utils/css/item-popup`. A helper, `getItemPopupWidthClassName( popupWidth )`, maps a preset to a CSS-module class. That file's contents are not in the visible (truncated) diff; per the PR description, fixed presets use explicit `width` and `content` allows content-sized width between anchor and available viewport bounds.

Each of the three `Popup` components (`autocomplete/popup.tsx`, `combobox/popup.tsx`, `select/popup.tsx`) destructures `popupWidth` and adds the resulting class between `itemPopupStyles.popup` and the consumer `className`:

```tsx
className={ clsx(
	itemPopupStyles.popup,
	getItemPopupWidthClassName( popupWidth ),
	className
) }
```

The `*PopupProps` types in each `types.ts` are intersected with `ItemPopupWidthProps`, as is `SearchableChipSelectProps`. `SearchableChipSelect` forwards `popupWidth` to `Combobox.Popup`. `SelectControl` uses `popupWidth = 'content'` and forwards it to `Select.Popup`.

In `@wordpress/block-editor`, `position.jsx` adds `popupWidth="anchor"` to the control, drops the `className` from its items, and deletes the `max-inline-size` rule from `block-hooks.scss`.

Storybook: `PopupWidth` stories are added for `SelectControl` and `SearchableChipSelectControl`. The `Creatable` story is refactored to take `items` and `value` from args, and shared helpers `longLabelPopupItems` and `narrowContainerDecorator` are added. `packages/ui/CHANGELOG.md` records both a Breaking Changes entry and an Enhancements entry.

## Contribution

Closes issue #82014. A reviewer asked whether `Menu` should get the same presets, given the planned convergence of `Menu` styles with the other item popups. @mirka replied that `Menu` typically has a very narrow icon-button anchor and a smaller content range, so it would need a different default and may not need the flexibility; the question was left to the reviewer's judgment. @mirka also checked remaining in-repo usages. Storybook usages were fine, and the `location-picker` field type had a pre-existing layout issue deferred to #82090.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
