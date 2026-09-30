# #83261: Icons: Add a core-admin collection for the admin interface

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`dd9dedc`](https://github.com/WordPress/gutenberg/commit/dd9dedc58d33a963828b8d5a60f96760ed35ba48)
- **Discussion:** [#83261](https://github.com/WordPress/gutenberg/pull/83261) · 18 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds a `core-admin` icon collection to the WordPress icons registry, separating the icons the admin bar and admin menu render from the public `core` collection. The manifest's `public` boolean is replaced by a `collections` array, so each icon explicitly lists which collections it belongs to. The `core-admin` collection is registered with `public => false`, meaning its icons are invisible to the REST API and the Icon block. This prevents a consumer from accidentally breaking admin SVG icons by unregistering the `core` collection.

## Impact

- **Plugin & theme developers (icons manifest):** The `public` field in `manifest.json` is replaced by a `collections` array. Icons that previously had `"public": true` now use `"collections": ["core"]`. Icons that were non-public but needed server-side (e.g. `brush`, `dashboard`) now use `"collections": ["core-admin"]`. Update any custom icon manifests accordingly.
- **Plugin & theme developers (unregistering `core`):** Calling `wp_unregister_icon_collection( 'core' )` no longer removes SVG icons from the admin bar or admin menu, because those icons now live in `core-admin` as well.
- **Headless & REST consumers:** `/wp/v2/icon-collections` now returns a `core-admin` entry, but `/wp/v2/icons/core-admin` returns an empty array. The Icon block's picker omits the collection entirely (same behavior as empty pattern categories).
- **No action required** for sites that do not interact with the icons manifest or the icons REST API directly.

## Technical details

**Collection registration (`lib/icons.php`):** `gutenberg_register_default_icon_collections()` now calls `wp_register_icon_collection( 'core-admin', array( 'label' => ..., 'description' => ..., 'public' => false ) )` alongside the existing `core` registration. The `public => false` flag at the collection level is what keeps the collection's icons out of the REST API.

**Icon registration loop (`lib/icons.php`):** `gutenberg_register_default_icons()` previously registered every manifest icon as `core/{slug}`. It now iterates `$icon_data['collections']` and calls `wp_register_icon( $collection_slug . '/' . $icon_name, $icon_args )` for each listed collection. A `_doing_it_wrong()` fires if an icon entry lacks a non-empty `collections` array.

**Registry upgrade path (`lib/class-wp-icons-registry-gutenberg.php`):** `get_instance()` now skips both `core/` and `core-admin/` prefixed icons when replaying icons from the original `WP_Icons_Registry` instance, so the Gutenberg registry does not double-register them.

**Manifest schema (`packages/icons/src/manifest.json`):** Every icon that ships to core now carries a `collections` array instead of a `public` boolean. Examples from the diff:

```json
// Before
{ "slug": "home", "label": "Home", "filePath": "library/home.svg", "public": true }

// After (icon in both collections)
{ "slug": "home", "label": "Home", "filePath": "library/home.svg", "collections": ["core", "core-admin"] }

// After (admin-only icon, previously had no `public` field)
{ "slug": "dashboard", "label": "Dashboard", "filePath": "library/dashboard.svg", "collections": ["core-admin"] }
```

Icons in `core-admin` only (e.g. `brush`, `dashboard`, `link`, `media`, `page`, `pin`, `plugins`, `sites`, `tool`, `update`, `wordpress`) are registered server-side but never exposed via REST or the Icon block. Icons in both `core` and `core-admin` (e.g. `chevron-left`, `comment`, `home`) are public and also available to the admin interface.

**Build tooling (`packages/icons/lib/generate-manifest-php.cjs`):** The generated `manifest.php` now includes a `collections` array per icon. The filter for which icons ship to core changed from `item.public === true` to `item.collections?.length > 0`.

**Validation (`packages/icons/lib/validate-collection.cjs`):** Replaces the old `public`-is-boolean check with a check that `collections` is a non-empty array of unique slugs drawn from `['core', 'core-admin']`.

## Contribution

Opened by @t-hamano as an alternative to #83012, motivated by the need to support wordpress-develop#12270 (menu API accepting icon-registry names). During review, @fushar proposed moving the `public` concept from the icon level to the collection level (`wp_register_icon_collection()` accepting `public => false`), arguing it simplifies the mental model and eliminates the need for the Icon block's custom inserter to filter out empty collections. @t-hamano and @mcsf agreed, and the PR was updated accordingly before merge. The PR depended on #83277 (already merged) and was coordinated with a follow-up PR (#83338) to finalize which SVG icons the admin bar and sidebar would actually use.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
