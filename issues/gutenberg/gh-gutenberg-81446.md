# #81446: components/Menu: Restore Modal focus return when menu items close (#81164)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Components`
- **Merged:** [`7e91469`](https://github.com/WordPress/gutenberg/commit/7e91469771fab01b0c98f5aeb4bbf2110530db5b)
- **Discussion:** [#81446](https://github.com/WordPress/gutenberg/pull/81446) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Backport to the `wp/7.1` branch of Gutenberg #81164. It restores focus return to the root menu trigger when a `Modal` opened from a `Menu` item is closed, while keeping body scroll locked during the menu-to-Modal handoff. The same change also makes `Modal` prevent background scrolling without depending on WordPress global styles when the default `bodyOpenClassName` is used. The cherry-pick needed manual conflict resolution in three non-logic files.

## Impact

- **Plugin & theme developers using `@wordpress/components` `Menu`:** If a `Menu.Item` opens a `Modal`, closing the Modal now returns focus to the menu trigger (the `Open menu` button). Previously focus was lost. No API changes and no code changes are required.
- **Accessibility:** Keyboard and screen-reader users keep their place after dismissing a Modal launched from a menu. The PR author smoke-tested this with Windows NVDA.
- **Developers who worked around this:** Code that manually re-focused the trigger, or that set `hideOnClick={ false }` and a raised `z-index` on the overlay to stack a Modal over the menu, may no longer need to. The updated story drops that pattern.
- **Behavior to be aware of:** `hideOnClick` on `Menu.Item`, `Menu.CheckboxItem` and `Menu.RadioItem` is now routed through a shared hook, so its close behavior is computed rather than passed straight to Ariakit. Function-valued `hideOnClick` callbacks and `event.preventDefault()` in `onClick` are covered by new tests.
- **Site owners / hosting:** No action required.

## Technical details

**New hook.** The diff adds `use-menu-item-hide-on-click` (`useMenuItemHideOnClick( store, hideOnClick )`) in `packages/components/src/menu/`. Its implementation is not visible in the truncated diff, so its internals are not described here.

**Wiring.** `item.tsx`, `checkbox-item.tsx` and `radio-item.tsx` now call the hook and pass the result to the Ariakit-based `Styled.*` item as `hideOnClick={ computedHideOnClick }`, replacing the raw `hideOnClick` prop.

```tsx
// before
<Styled.Item hideOnClick={ hideOnClick } store={ computedStore } />

// after
const computedHideOnClick = useMenuItemHideOnClick( computedStore, hideOnClick );
<Styled.Item store={ computedStore } hideOnClick={ computedHideOnClick } />
```

In `Item`, `computedStore` (`store ?? menuContext?.store`) is now computed before the guard, which throws if `menuContext?.store` or `computedStore` is missing.

**Modal scroll lock.** Per the CHANGELOG, `Modal` now prevents background scrolling without relying on WordPress global styles when the default `bodyOpenClassName` is used. The Modal source changes fall in the truncated part of the diff.

**Story.** `WithModals` (outer and inner modals, `hideOnClick={ false }`, Emotion `useCx` z-index override) is replaced by a simpler `WithModal` story with a default `Menu.Item` that opens one `Modal`.

**Tests** (`menu/test/index.tsx`): new cases cover focus return to the root trigger after pointer and Enter activation, after a nested submenu opens a Modal, and scroll-lock handoff (`document.body.style.overflow === 'hidden'` or the `modal-open` class). Other cases cover `hideOnClick` booleans and callbacks, checkbox/radio items, focus moved outside by a click handler or callback, and prevented activation.

**Backport conflicts:** only `CHANGELOG.md`, `stories/index.story.tsx` and `tools/eslint/suppressions.json`. The description states no component logic changed.

## Contribution

This is a manual backport of #81164 to `wp/7.1`, opened as a dedicated PR because the automated cherry-pick did not apply cleanly. Claude Code was used to resolve the conflicts and draft the description, and the author reviewed the diff locally. A flaky RTC collaboration e2e test was reported but is unrelated to the change. The only human comment is the author's NVDA smoke-test confirmation; props-bot credits `ciampo` and `t-hamano`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
