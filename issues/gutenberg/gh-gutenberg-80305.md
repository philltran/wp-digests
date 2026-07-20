# #80305: Try fixing responsive layout in Nav block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Navigation`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`1222f40`](https://github.com/WordPress/gutenberg/commit/1222f40a9863647c137344caaf29b1c01724c62c)
- **Discussion:** [#80305](https://github.com/WordPress/gutenberg/pull/80305) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Viewport-specific layout styles in the Navigation block were not applying correctly because layout custom properties and state classes were duplicated across outer and inner containers. This PR fixes the issue by generating viewport-specific CSS custom properties and scoping them to the inner container using a unique class. The change ensures responsive layout adjustments (orientation, justification, flex-wrap) work reliably in both the editor and on the front end.

## Impact

- **Site owners & editors**: Responsive layout settings for the Navigation block now apply correctly across viewports.
- **Plugin & theme developers**: The internal render filter callback `block_core_navigation_add_state_class_to_container` was renamed to `block_core_navigation_add_support_classes_to_container`. If you hooked into `render_block` and targeted the old function name, update your references.
- **No action required** for most users; this is a transparent fix to block rendering logic.

## Technical details

The fix spans PHP rendering and JS editor logic:
- **PHP** (`packages/block-library/src/navigation/index.php`): Introduces `block_core_navigation_get_layout_custom_property_declarations()` to map layout attributes to CSS custom properties (`--navigation-layout-direction`, `--navigation-layout-justify`, `--navigation-layout-align`, `--navigation-layout-wrap`, `--navigation-layout-justification-setting`). The render filter callback is renamed to `block_core_navigation_add_support_classes_to_container()`. It reads viewport breakpoints from the `style` attribute, merges them with the base `layout` attribute, and generates scoped CSS rules via `wp_style_engine_get_stylesheet_from_css_rules()`. A unique class (generated via `wp_unique_id( 'wp-block-navigation-' )`) is injected into the inner `wp-block-navigation__container` to scope the custom properties, preventing cascade conflicts.
- **JS/Editor** (`packages/block-library/src/navigation/edit/use-layout-custom-properties.js`): Adds a new `useLayoutCustomProperties` hook that mirrors the PHP logic for the editor canvas. It uses `useStyleOverride` to inject viewport-specific CSS custom properties scoped to `#block-{clientId}`.
- The approach avoids modifying the core layout support system, opting instead for a scoped workaround similar to how the Gallery block handles responsive custom styles.

## Contribution

Opened by @tellthemachines as part of #77817 and merged with co-authorship from @talldan. The author noted they briefly considered upgrading Nav to use default layout logic targeting the correct wrapper but rejected it as non-trivial due to Nav's reusable block nature and backward compatibility constraints. The fix was backported to the `wp/7.1` branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
