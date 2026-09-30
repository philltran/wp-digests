# #83185: Submenu: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Submenu`
- **Merged:** [`12b1d36`](https://github.com/WordPress/gutenberg/commit/12b1d364d144a2970b4838db60735fb2b441f84a)
- **Discussion:** [#83185](https://github.com/WordPress/gutenberg/pull/83185) · 8 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The Submenu block (`core/navigation-submenu`) now supports the `shadow` block support, applying a `box-shadow` to the dropdown panel (`.wp-block-navigation__submenu-container`) rather than to the menu item itself. The shadow is available through Global Styles and per-block overrides, with responsive variants, and is correctly suppressed when the submenu renders inline (overlay mode or always-open). This closes a gap where the floating dropdown layer had no way to receive a shadow via the standard design tools.

## Impact

- **Theme & site builders:** A new Shadow control appears under Blocks → Submenu in Global Styles and in the block inspector. No code changes required; the feature is opt-in via the existing shadow presets.
- **Block & theme developers:** The `core/navigation-submenu` block now declares `"shadow": { "__experimentalSkipSerialization": true }` in its `supports` and a `selectors.shadow` entry. If you build custom navigation markup or extend this block, the shadow is applied as an inline style on the dropdown `ul`, not on the `li` wrapper. The `!important` resets in `style.scss` for overlay and always-open contexts mean a plain `box-shadow: none` without `!important` will not override a block-level shadow in those modes.
- **No breaking changes.** Existing Submenu blocks render identically until a shadow is explicitly set.

## Technical details

**`block.json`** adds two keys:

```json
"supports": {
  "shadow": { "__experimentalSkipSerialization": true }
},
"selectors": {
  "shadow": ".wp-block-navigation-submenu > .wp-block-navigation__submenu-container"
}
```

`__experimentalSkipSerialization` prevents the Style Engine from emitting `box-shadow` on the `li` wrapper (both the `li` and the inner `ul` call `get_block_wrapper_attributes()`). The `selectors` entry redirects Global Styles shadow rules to the dropdown container only, avoiding leakage onto Navigation Link and Page List dropdowns that share the `.wp-block-navigation__submenu-container` class.

**`index.php`** (`render_block_core_navigation_submenu`): after the existing color-style handling, the function now checks `$attributes['style']['shadow']` and, if present, calls `wp_style_engine_get_styles( array( 'shadow' => $attributes['style']['shadow'] ) )`, appending the resulting CSS to the `$style_attribute` string that is written as an inline `style` on the dropdown `ul`.

**`edit.jsx`**: imports `__experimentalGetShadowClassesAndStyles` (aliased as `getShadowClassesAndStyles`) from `@wordpress/block-editor`, calls it with `attributes`, and merges the returned `style` object into the props passed to `useInnerBlocksProps`, so the editor dropdown matches the front end.

**`style.scss`** adds two `box-shadow: none !important` resets:
- Inside the overlay-menu context (`.wp-block-navigation__responsive-container-open`), alongside the existing `border`, `background-color`, and `color` resets.
- Inside the always-open submenu context, targeting `.wp-block-navigation-item .wp-block-navigation__submenu-container`.

Both use `!important` because the block-level shadow is an inline style on the dropdown element, which would otherwise win over a stylesheet rule.

## Contribution

Opened by @aaronrobertshaw, who noted uncertainty about whether the shadow should target the dropdown or the menu item and flagged it for review. @ramonjd endorsed the dropdown-only approach. @andrewserong identified during review that in the Navigation block's overlay mode the submenu renders statically rather than as a popover, and asked whether the shadow should be reset there. @aaronrobertshaw confirmed the oversight and pushed a follow-up commit adding `box-shadow: none !important` resets for both the overlay and always-open contexts. The PR was implemented with the assistance of a Claude Code agent. Merged as `12b1d36`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
