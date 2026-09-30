# #81164: components/Menu: Restore Modal focus return when menu items close

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Components`, `Backported to WP Core`
- **Merged:** [`13a0338`](https://github.com/WordPress/gutenberg/commit/13a0338a03ae579ee9298ee99596cb07f789ddee)
- **Discussion:** [#81164](https://github.com/WordPress/gutenberg/pull/81164) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a regression in `@wordpress/components` where closing a `Menu` item that opens a legacy `Modal` no longer returned focus to the menu's root trigger button when the Modal closed. Before the menu unmounts, focus is now moved to the root trigger if it is still inside the menu, so `Modal` captures a valid focus-return target. `Modal` also now ships the default `body.modal-open { overflow: hidden; }` rule itself, so scroll lock no longer depends on WordPress `common.css`.

## Impact

**Plugin & theme developers**
- Anyone opening a `Modal` from a `Menu.Item`, `Menu.CheckboxItem`, or `Menu.RadioItem` gets focus back on the root menu trigger when the Modal closes (a11y regression fix from #77460).
- `Modal` used outside wp-admin (Storybook, standalone apps, front-end usage without `common.css`) now locks body scroll with the default `bodyOpenClassName`.
- If you pass a custom `bodyOpenClassName`, you still have to supply your own scroll-lock CSS; the JSDoc now says so explicitly.
- Existing `hideOnClick` behavior (boolean or callback) is unchanged. Items that call `event.preventDefault()` in `onClick` still keep the menu open.

**Site owners / others**
- No action required. No API removals or deprecations.

**Core**
- Tagged `Backported to WP Core` for the wp/7.1 branch; the automated cherry-pick conflicted and a manual PR (#81446) was opened.

## Technical details

**Menu items.** A new hook, `useMenuItemHideOnClick( store, hideOnClick )` in `packages/components/src/menu/use-menu-item-hide-on-click.ts`, is used by `Menu.Item` (`item.tsx`), `Menu.CheckboxItem` (`checkbox-item.tsx`) and `Menu.RadioItem` (`radio-item.tsx`). Each now passes the computed value as `hideOnClick` to the Ariakit item instead of the raw prop. The hook file is truncated in the provided diff, so its exact body isn't shown. Per the PR description, when an item is about to close its menu and focus is still inside it, focus moves to the root menu button before Ariakit unmounts the menu. The immediate-unmount teardown from #77460 and the Ariakit activation safeguards are kept.

In `Item`, `computedStore` (`store ?? menuContext?.store`) is now resolved before the guard so it can be passed to the hook; the guard also checks `! computedStore`.

**Modal.** The description says `Modal` now includes the default `body.modal-open { overflow: hidden; }` style, and the bundle-size report shows small increases in `components/style*.css`. The `Modal` source diff is not visible in the truncated diff. The `bodyOpenClassName` JSDoc now states that custom class names need their own scroll-lock styles.

**Tests/stories.** New Jest tests cover focus return after pointer and Enter activation, nested submenus, scroll-lock handoff from Menu to Modal, `hideOnClick` callback and boolean paths for checkbox and radio items, focus moved outside by handlers, and prevented activation. The `WithModals` story is replaced by a simpler `WithModal` story with no `hideOnClick={ false }` or z-index overrides. Two CHANGELOG bug-fix entries are added.

## Contribution

Closes #80734, a regression traced to #77460, which made menus unmount immediately to preserve the scroll-lock handoff at the cost of Modal's focus-return target. In review, a tester found that scroll lock in the Storybook example only worked with WordPress `common.css` injected. @ciampo agreed it was pre-existing but added the default `overflow: hidden` to `Modal` and clarified the `bodyOpenClassName` docs. @t-hamano, running the 7.1 release, asked to target RC2. The automated backport to `wp/7.1` failed twice with conflicts, so @t-hamano opened #81446 by hand, noting the conflict was unrelated to the logic change. The author disclosed Codex was used to implement the change and tests.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
