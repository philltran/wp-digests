# Hiding the Classic block from the inserter in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Marin Atanasov
- **Published:** 2026-06-23
- **Tags:** `Core`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/06/23/hiding-the-classic-block-from-the-inserter-in-wordpress-7-1/](https://make.wordpress.org/core/2026/06/23/hiding-the-classic-block-from-the-inserter-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

In WordPress 7.1, the Classic block (`core/freeform`) is hidden from the block inserter by default. This change leaves all existing Classic blocks fully editable and untouched, while preventing new instances from being added through the inserter, block library, or slash commands. The shift aims to reduce legacy content creation and streamline editor architecture without breaking backward compatibility.

## Impact

- Plugin & theme developers: If your code programmatically inserts `core/freeform` into the inserter registry, it will now be hidden unless you hook `wp_classic_block_supports_inserter`.
- Site owners & editors: New Classic blocks will not appear in the default inserter; existing content remains editable and renders identically.
- Opt-out path: Add a single filter or use a dedicated plugin to restore visibility. No database migration or content modification occurs.

## Technical details

A new public filter, `wp_classic_block_supports_inserter`, governs whether `core/freeform` appears in the inserter UI. It receives two parameters: a boolean `$supports_inserter` flag and the current `$post` object (WP_Post). The implementation does not alter block registration, render logic, or edit modes; it only toggles inserter visibility. Example opt-in pattern:
```php
add_filter( 'wp_classic_block_supports_inserter', '__return_true' );
// or conditionally per post type:
add_filter( 'wp_classic_block_supports_inserter', function( $supports, $post ) {
    return 'page' === $post->post_type ? true : $supports;
}, 10, 2 );
```
The block remains registered under `core/freeform`. References: Trac #11712, #65166; Gutenberg Issue #77911.

## Contribution

Merged for the WordPress 7.1 release cycle following discussion across Make/Core and Gutenberg repositories. Key contributors include @desrosj, @mamaduka, @mukesh27, @westonruter, @wildworks, and @yuliyan for code review and architecture guidance. The implementation followed a simple filter-based opt-out model to avoid modifying core block registration or UI injection logic, aligning with the broader goal of gradually deprecating legacy `core/freeform` workflows.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
