# #83277: Icons: Move the public property from icons to collections

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`a002558`](https://github.com/WordPress/gutenberg/commit/a00255831999ba4509ae3835a65029ea1acd9008)
- **Discussion:** [#83277](https://github.com/WordPress/gutenberg/pull/83277) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The icon registry's `public` visibility flag moves from individual icons to icon collections. `wp_register_icon_collection()` now accepts a `public` argument (default `true`); collections marked non-public are excluded from the `/wp/v2/icon-collections` and `/wp/v2/icons` REST endpoints, while their icons remain retrievable via `wp_get_icon()`. The icon-level `public` property introduced in #82634 is removed, and the tri-state `manifest.json` behavior from that PR is reverted. This is a prerequisite for the separate `core-admin` collection proposed in #83261.

## Impact

- **Plugin & theme developers:** The icon-level `public` argument to `wp_register_icon()` is removed. If you registered icons with `'public' => false` (from #82634, which never shipped in a Gutenberg plugin release), you must instead register a non-public collection via `wp_register_icon_collection( $slug, array( 'public' => false, ... ) )` and register your icons under that collection.
- **Headless & REST consumers:** Icons belonging to a non-public collection return `rest_icon_not_found` (404) from `/wp/v2/icons` and `rest_icon_collection_not_found` (404) from `/wp/v2/icon-collections`. No change for collections registered without the `public` key (they default to `true`).
- **Hosting & platform:** No configuration or migration required. The change is additive at the collection level; existing public collections behave identically.
- **No action required** for sites that do not use the icon registry's `public` flag or the icon REST endpoints directly.

## Technical details

A new class `WP_Icon_Collections_Registry_Gutenberg` (in `lib/class-wp-icon-collections-registry-gutenberg.php`) extends `WP_Icon_Collections_Registry` and overrides `register()` to extract and store a `public` boolean (default `true`) on each collection entry. Its `get_instance()` migrates any already-registered collections from the parent class. A hook on `init` at priority `-1` (`gutenberg_override_wp_icon_collections_registry`) swaps the shared instance before default collections register.

In `lib/class-wp-icons-registry-gutenberg.php`, the `public` key is removed from `$allowed_keys` in `register()`, the boolean validation block is deleted, and the manifest-loading path in `get_instance()` no longer copies `$icon['public']` into icon properties.

A new `WP_REST_Icon_Collections_Controller_Gutenberg` (in `lib/class-wp-rest-icon-collections-controller-gutenberg.php`) overrides `get_items()` to skip non-public collections and `get_item()` to return a 404 `WP_Error` for non-public slugs. The existing `WP_REST_Icons_Controller_Gutenberg` in `lib/class-wp-rest-icons-controller-gutenberg.php` is updated: `get_items()` now checks the *collection's* `public` flag instead of the icon's, and `get_icon()` resolves the icon's collection and returns `rest_icon_not_found` if that collection is non-public.

The old collections-controller registration in `lib/compat/wordpress-7.1/rest-api.php` is removed; the new one in `lib/rest-api.php` instantiates `WP_REST_Icon_Collections_Controller_Gutenberg`.

In `lib/icons.php`, the block that copied `$icon_data['public']` into per-icon registration args is deleted. In `packages/icons/lib/generate-manifest-php.cjs`, the tri-state logic (omitted / `true` / `false`) is simplified: only icons with `public: true` are emitted into `manifest.php`, and the generated PHP no longer includes a `'public'` key.

Before (icon-level, #82634):
```php
wp_register_icon( 'core/dashboard', array(
    'label'     => __( 'Dashboard', 'gutenberg' ),
    'file_path' => $path,
    'public'    => false,
) );
```
After (collection-level):
```php
wp_register_icon_collection( 'core-admin', array(
    'label'  => __( 'Core Admin', 'gutenberg' ),
    'public' => false,
) );
wp_register_icon( 'core-admin/dashboard', array(
    'label'     => __( 'Dashboard', 'gutenberg' ),
    'file_path' => $path,
) );
```

## Contribution

Opened by @t-hamano as a prerequisite for #83261 (the `core-admin` collection). @fushar suggested folding the `core-admin/` registration into this PR to simplify review and testing, but @t-hamano preferred to keep the two enhancements in separate commits. @t-hamano flagged that the PR needed to land in Gutenberg 24.1 to prevent the icon-level `public` option from being exposed in a plugin release; it was merged as `a002558` with that timing in mind, with a backport changelog entry targeting 7.2.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
