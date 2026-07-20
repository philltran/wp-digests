# Consistent navigation in WordPress 7.1 with persistent toolbar

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Ashar Fuadi
- **Published:** 2026-07-13
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`, `editor`
- **Link:** [https://make.wordpress.org/core/2026/07/13/consistent-navigation-in-wordpress-7-1-with-persistent-toolbar/](https://make.wordpress.org/core/2026/07/13/consistent-navigation-in-wordpress-7-1-with-persistent-toolbar/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces a persistent admin toolbar to the Post and Site Editors, replacing the ambiguous “W” logo back button with a dedicated chevron and displaying the site icon in the toolbar. This change standardizes navigation across the entire admin interface, ensuring the “W” logo consistently opens the About page, the site icon opens the site menu, and the chevron returns to the previous screen. The toolbar remains hidden only when Distraction Free mode is active.

## Impact

- **Plugin & theme developers:** Plugins that add `WP_Admin_Bar` nodes will now render in the Site Editor and Post Editor by default. Developers should verify compatibility and conditionally hide nodes if they are not intended for editor contexts.
- **Site owners & editors:** Navigation in the editor is now consistent with the rest of the admin. The toolbar is visible by default; users can hide it via Distraction Free mode.
- **No action required** for core functionality, but plugin authors should audit toolbar integrations to prevent unexpected UI overlap in the editor.

## Technical details

The change modifies the editor’s navigation layer to render the `WP_Admin_Bar` by default in both the Post and Site Editors, excluding Distraction Free mode. The previous “W” logo back button is replaced with a chevron, and the site icon is now rendered in the toolbar. Plugin developers can conditionally suppress toolbar nodes using the `admin_bar_menu` action and checking the current screen:

```php
add_action(
    'admin_bar_menu',
    function ( WP_Admin_Bar $wp_admin_bar ) {
        $screen = function_exists( 'get_current_screen' ) ? get_current_screen() : null;

        // use this check to hide the node in the Site Editor
        if ( $screen && 'site-editor' === $screen->id ) {
            return;
        }
        // ... or use this check to hide the node in any block editor
        if ( $screen && $screen->is_block_editor() ) {
            return;
        }

        $wp_admin_bar->add_node( ... );
    },
    100
);
```

The `WP_Admin_Bar` class and `admin_bar_menu` hook are the primary interfaces affected. The `get_current_screen()` function and `$screen->is_block_editor()` method are recommended for conditional rendering.

## Contribution

This change was developed as part of the WordPress 7.1 editor initiative, addressing navigation inconsistencies tracked in Trac tickets #65091 and #65088. The implementation standardizes the toolbar behavior across editor contexts, replacing the legacy logo-based navigation with explicit back and site-menu controls. Review and refinement were provided by @tyxla, @mayanktripathi32, @joen, @lucasmdo, @mamaduka, and @annezazu.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
