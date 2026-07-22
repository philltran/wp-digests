# #80319: Update view config API versioning

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Bug`, `Backported to WP Core`
- **Merged:** [`eab22e1`](https://github.com/WordPress/gutenberg/commit/eab22e163b1f1875f712ff4002decd242969d070)
- **Discussion:** [#80319](https://github.com/WordPress/gutenberg/pull/80319) · 12 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `Gutenberg_View_Config_Data` class used by the `get_entity_view_config_{$kind}_{$name}` filter has been refactored to replace shape-specific mutation methods with a unified patch-based API. The previous `update_properties()`, `update_view_list_items()`, and `update_form_fields()` methods are replaced by `merge()`, `replace()`, `set()`, and `remove()`, all of which accept a `$patch` (or `$spec`) and a `$version` argument. This change standardizes how developers modify DataViews configuration for entity lists and forms, making patches composable and future-proof against schema changes.

## Impact

- **Plugin & theme developers:** Code using the old `update_properties()`, `update_view_list_items()`, or `update_form_fields()` methods will break. Developers must migrate to the new `merge()`, `replace()`, `set()`, or `remove()` methods.
- **No action required** for developers who do not interact with the `get_entity_view_config_{$kind}_{$name}` filter or DataViews configuration.
- The `get_config()` method is now private (`get_data()`), preventing direct access to the materialized configuration array and enforcing patch-based mutations.

## Technical details

The diff replaces the four shape-specific methods in `lib/compat/wordpress-7.1/class-gutenberg-view-config-data.php` with a unified gradient of operations:
- `merge( $patch, $version )`: Recursively applies a patch. Maps merge key-by-key, lists merge by identity (`id`, `slug`, or `field`), and `null` deletes a leaf.
- `replace( $patch, $version )`: Same as `merge()`, but any list in the patch replaces the current list wholesale instead of merging by identity.
- `set( $patch, $version )`: Swaps each named top-level key wholesale, dropping inherited defaults for those keys.
- `remove( $spec, $version )`: Deletes entries by name spec rather than value patch, mirroring the config shape to prune specific members without touching the rest.

The old `get_config()` method is renamed to `get_data()` and marked `private`. The class now exposes an `apply_filters( $kind, $name )` method that runs the dynamic filter and reconciles the result. Core's per-post-type callbacks are migrated to the new methods, and documentation in `docs/how-to-guides/curating-the-editor-experience/filters-and-hooks.md` is updated.

**Before:**
```php
$data->update_view_list_items( array( 'drafts' => null ), 1 );
$data->update_form_fields( array( 'slug' => null ), 1 );
```

**After:**
```php
$data->remove( array( 'view_list' => array( 'drafts' ) ), 1 );
$data->merge( array( 'form' => array( 'fields' => array( 'title', 'status' ) ) ), 1 );
```

## Contribution

During review, the team debated whether to rely on `null` values for deletion or introduce a dedicated removal method, and whether to support list reordering. The author proposed adding `set()` and `remove()` to handle wholesale swaps and targeted pruning, while reviewers emphasized making `get_data()` private to prevent consumers from reading the materialized config. The team ultimately merged the four-method gradient, explicitly rejecting list reordering to preserve future schema flexibility, and updated the documentation to reflect the new patch model.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
