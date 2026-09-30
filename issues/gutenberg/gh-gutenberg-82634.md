# #82634: Icons: Allow icons to ship to WordPress Core without exposing them in the Icon block 

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`, `[Package] Icons`
- **Merged:** [`9be14bd`](https://github.com/WordPress/gutenberg/commit/9be14bd021d7f4eec81d0df6948c90b7c4588b52)
- **Discussion:** [#82634](https://github.com/WordPress/gutenberg/pull/82634) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `public` property in the icons `manifest.json` is now a tri-state rather than a simple boolean. Omitting it keeps an icon in the JS library only; `true` ships it to Core and exposes it through the icons REST API and Icon block; `false` ships it to Core and registers it for server-side use via `wp_get_icon()` while hiding it from the REST API and the Icon block. The `core/wordpress` icon is the first to use `public: false`, making it available to admin-screen code without appearing as a selectable icon in post content.

## Impact

- **Plugin & theme developers:** `wp_register_icon()` (and the Gutenberg-side `WP_Icons_Registry_Gutenberg::register()`) now accepts a `public` boolean argument. Omitting it preserves existing behavior (icon is public). Passing `false` registers the icon in the `core` collection for `wp_get_icon()` but excludes it from the `/wp/v2/icons` REST routes. Passing a non-boolean triggers `_doing_it_wrong()` (since 7.2.0) and the registration is rejected.
- **Headless & REST consumers:** Icons registered with `public: false` are omitted from `GET /wp/v2/icons` and return a 404 (`rest_icon_not_found`) on `GET /wp/v2/icons/{name}`. No change for icons that are `public: true` or have no `public` key.
- **Hosting & platform:** No configuration or migration required. The change is additive; existing icon registrations are unaffected.
- **No action required** for sites that do not register custom icons or consume the icons REST API.

## Technical details

**Build pipeline.** `packages/icons/lib/generate-manifest-php.cjs` previously filtered `manifest.json` entries with `item.public === true`. It now filters with `typeof item.public === 'boolean'`, so both `true` and `false` entries are emitted into `manifest.php`. For `false` entries it appends `'public' => false` to the generated PHP array. The `bin/build-plugin-zip.sh` script's `jq` filter changed from `select(.public)` to `select(.public != null)` so that `public: false` SVGs are retained in Core builds rather than deleted.

**Registry.** `WP_Icons_Registry_Gutenberg::register()` adds `public` to its `$allowed_keys` array and validates it is a boolean (rejecting with `_doing_it_wrong()` otherwise). The static `get_instance()` method now copies `$icon['public']` into the properties passed to `register()` when present. The compat shim `wp_register_icon()` in `lib/compat/wordpress-7.1/icons.php` documents the new parameter and `gutenberg_register_default_icons()` forwards the `public` key from manifest data.

**REST controller.** `WP_REST_Icons_Controller_Gutenberg::get_items()` adds an early `continue` when `( $icon['public'] ?? true )` is `false`. A new `get_icon()` override calls `parent::get_icon( $name )` and, if the result is not a `WP_Error` and the icon's `public` is `false`, returns `new WP_Error( 'rest_icon_not_found', …, array( 'status' => 404 ) )`.

**Validation.** `packages/icons/lib/validate-collection.cjs` now checks that `public`, if present in a manifest entry, is a boolean and reports a problem string otherwise.

**Proof-of-concept entry.** `packages/icons/src/manifest.json` gains `"public": false` on the `wordpress` slug, and the generated `manifest.php` gains:

```php
'wordpress' => array(
    'label'    => _x( 'WordPress', 'icon label', 'gutenberg' ),
    'filePath' => 'library/wordpress.svg',
    'public'   => false,
),
```

**Tests.** New PHPUnit cases verify that `core/wordpress` is absent from `GET /wp/v2/icons` and returns 404 on direct fetch, and that registering an icon with a non-boolean `public` triggers `_doing_it_wrong` and fails. Existing test `set_up()` methods were updated to reference `WP_Icons_Registry_Gutenberg` instead of `WP_Icons_Registry`.

## Contribution

Opened by @fushar as part of issue #82723, implementing the approach suggested in the earlier PR #79451. Co-authored with @scruffian and @tyxla. The author noted use of Claude Code Opus for drafting and reviewing the changes. The PR was merged by @fushar with no recorded design debate or rejected alternatives in the three comments on the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
