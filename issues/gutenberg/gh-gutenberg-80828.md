# #80828: [WP 7.1] Navigation: Fixes `aria-expanded` not updating on hover submenu inside overlay

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Package] Block library`, `[Block] Navigation`
- **Merged:** [`82a0932`](https://github.com/WordPress/gutenberg/commit/82a0932ec93f7273c8007961c280bf1686f8ca8f)
- **Discussion:** [#80828](https://github.com/WordPress/gutenberg/pull/80828) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request fixes an accessibility bug in the Navigation block where the `aria-expanded` attribute fails to update when hovering over a submenu inside an overlay menu. The change introduces a new `isSubmenuOpen` state in the Interactivity API store and updates the PHP template to bind `aria-expanded` to this state instead of the generic `isMenuOpen`. This ensures assistive technologies correctly announce the expanded state of nested menus when the parent overlay is active.

## Impact

- **Theme & plugin developers:** No breaking changes or migration steps required. The fix is internal to the Navigation block's view script and PHP template.
- **Site owners & editors:** Improves screen reader accuracy for sites using the Navigation block with overlay menus and hover-activated submenus.
- **Headless & REST consumers:** No impact; this is a frontend view-layer fix.
- **Action required:** None. The change ships automatically with WordPress 7.1.

## Technical details

The diff modifies three files in `packages/block-library/src/navigation/`:
- `index.php`: Changes the directive binding from `data-wp-bind--aria-expanded="state.isMenuOpen"` to `data-wp-bind--aria-expanded="state.isSubmenuOpen"`.
- `view.js`: Adds a computed `isSubmenuOpen` getter to the Interactivity store. It returns `true` if `state.isMenuOpen` is true or if any value in `ctx.overlayOpenedBy` is truthy. It also simplifies `openMenuOnHover` and `closeMenuOnHover` by removing the conditional that previously skipped hover actions when the overlay was open.
- `test/view.js`: Adds a Jest test verifying that `submenuOpenedBy.hover` tracks correctly when the overlay is already open.

Before/after pattern in `view.js`:
```javascript
// Before
if ( type === 'submenu' && Object.values( overlayOpenedBy || {} ).filter( Boolean ).length === 0 ) {
    actions.openMenu( 'hover' );
}
// After
if ( type === 'submenu' ) {
    actions.openMenu( 'hover' );
}
```

## Contribution

This change is a manual backport to the `wp/7.1` branch after automatic label-based cherry-picks failed for forked repositories. A reviewer noted a follow-up fix for jsdom redundancy and suggested an automatic pick for it; the backport applied cleanly with no recorded design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
