# A unified public exposure flag for Abilities in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Milana Cap
- **Published:** 2026-08-04
- **Tags:** `General`, `#abilities-api`, `7.1`, `dev-notes`, `dev-notes-7-1`, `rest-api`
- **Link:** [https://make.wordpress.org/core/2026/08/04/a-unified-public-exposure-flag-for-abilities-in-wordpress-7-1/](https://make.wordpress.org/core/2026/08/04/a-unified-public-exposure-flag-for-abilities-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces a unified `meta['public']` flag for the Abilities API to signal that an ability is intended for external client consumption. This replaces the need to set channel-specific exposure flags like `show_in_rest` for every new integration, establishing a single source of truth for discoverability while preserving granular overrides. The change streamlines ability registration for REST, MCP adapters, and future AI/agent integrations without altering existing behavior or API signatures.

## Impact

- **Plugin & theme developers:** No immediate action required. Existing abilities using `meta['show_in_rest']` continue to work unchanged. Developers can optionally migrate to `meta['public'] => true` for abilities intended across multiple client types.
- **Headless & REST consumers:** No breaking changes. The REST API schema now includes the `public` boolean in ability metadata, but exposure logic remains identical.
- **Hosting & platform teams:** No configuration changes needed. The flag is purely metadata-driven and defaults to `false`.
- **Integration authors (MCP, WP-CLI, custom clients):** Should adopt the precedence rule (`$meta[$channel]['public'] ?? $meta['public'] ?? false`) when resolving exposure, and update any custom `wp_register_ability_args` filter logic to inherit the unified flag.

## Technical details

The change adds a `public` boolean to the resolved metadata array returned by `WP_Ability::get_meta()`, defaulting to `false`. When registering an ability via `wp_register_ability()`, setting `'meta' => array( 'public' => true )` automatically resolves `show_in_rest` to `true` for the REST API, following the precedence chain: `$show_in_rest = $meta['show_in_rest'] ?? $meta['public'] ?? false;`. Explicit channel-specific values (e.g., `show_in_rest => false`) override the general `public` flag. The REST API ability metadata schema is updated to expose the `public` field. Core abilities `core/get-site-info`, `core/get-user-info`, and `core/get-environment-info` were migrated to use `meta['public'] => true`.

**Before:**
```php
'meta' => array(
    'show_in_rest' => true,
),
```

**After:**
```php
'meta' => array(
    'public' => true,
),
```

Integrations can derive channel-specific metadata during registration using the existing `wp_register_ability_args` filter, following the same precedence rule: explicit channel value → inherited `public` → channel default.

## Contribution

Developed to address the maintenance overhead of duplicating exposure intent across channel-specific metadata as the Abilities API expands beyond REST, the implementation was merged in changeset [62729] with core ability migrations following in [62737]. The design was refined through Trac ticket #65568, with peer review provided by @gziolo and @benjamin_zekavica.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
