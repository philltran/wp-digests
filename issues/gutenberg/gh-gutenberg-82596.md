# #82596: Navigation: don't force submenus open inside a custom overlay

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @scruffian
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Navigation`
- **Merged:** [`f0bd7b3`](https://github.com/WordPress/gutenberg/commit/f0bd7b3024b20937ce9c1e404dbe40533081c920)
- **Discussion:** [#82596](https://github.com/WordPress/gutenberg/pull/82596) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Submenus inside a custom Navigation overlay were being forced open as soon as the overlay opened, ignoring the block's "Submenu Visibility" setting and leaving the submenu toggle inert. This fixes a regression from #77563, which made the toggle's `aria-expanded` reflect the default overlay's force-open behaviour but applied the binding to custom overlays too. The fix flags custom overlays in the Interactivity API context so submenus only report as open from the overlay when the default overlay is in use.

## Impact

- **Site owners / theme authors using custom navigation overlays:** Submenus in a custom overlay once again respect "Submenu Visibility" (click/hover). They start collapsed and open or close on interaction, instead of appearing expanded with no supporting CSS and no way to close them.
- **Default overlay users:** No change. Submenus still expand when the overlay opens, and the toggle reports `aria-expanded="true"`.
- **Plugin/theme developers:** No API changes, deprecations, or required migration. If you target the navigation interactivity state, note that submenu contexts now inherit a `hasCustomOverlay` flag from the responsive container.
- No action required beyond picking up the Gutenberg update.

## Technical details

The style that force-opens submenus is scoped to `:where(:not(.disable-default-overlay))`, so it never applies to a custom overlay. The `isSubmenuOpen` getter from #77563, however, returned true whenever any `overlayOpenedBy` value was set. In a custom overlay this made `[aria-expanded="true"] ~ .wp-block-navigation__submenu-container` expand every submenu.

The diff makes three changes:

- **`packages/block-library/src/navigation/index.php`**: in `get_responsive_container_markup()`, when `$has_custom_overlay` is true, the responsive container's directives get `wp_interactivity_data_wp_context( array( 'hasCustomOverlay' => true ) )` appended. This is derived from the same value that drives the `disable-default-overlay` class (whether the overlay template part actually rendered), so markup and styles cannot disagree. The container is an ancestor of every submenu, so the flag is inherited.
- **`packages/block-library/src/navigation/view.js`**: `isSubmenuOpen` now computes `isDefaultOverlayOpen = ! ctx.hasCustomOverlay && <any overlayOpenedBy value true>` and returns `isDefaultOverlayOpen || state.isMenuOpen`.
- **Tests**: a jsdom test for the custom-overlay case (closed until `openMenu( 'click' )`, closable via `closeMenu`), three PHPUnit tests in `class-wp-navigation-block-renderer-test.php` (default overlay, custom overlay, and an overlay template part that renders nothing), and two front-end e2e tests in `navigation-frontend-interactivity.spec.js`, one per overlay type.

The PHPUnit fallback test covers a deleted overlay or one belonging to an inactive theme. Such a part renders nothing, so no `disable-default-overlay` class is emitted and the default styles expand the submenus. The flag is therefore not set in that case, and the toggle state stays consistent with the styles. The bundle size change is +13 B on `navigation/view.min.js`.

## Contribution

The original PR by @scruffian threaded a `hasCustomOverlay` flag through `get_nav_attributes()` into `get_nav_element_directives()`, deriving it from `! empty( $attributes['overlay'] )`. @jeryj pushed a simpler replacement that sets the flag on the responsive container next to the existing `$has_custom_overlay`. The original condition could disagree with the `disable-default-overlay` class when the overlay part renders nothing. @scruffian agreed the revised approach was simpler and better. The PR notes the two e2e tests could not be run locally and relied on CI.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
