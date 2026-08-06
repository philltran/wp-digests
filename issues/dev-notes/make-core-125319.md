# Filtering registered abilities with wp_get_abilities() in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Milana Cap
- **Published:** 2026-08-05
- **Tags:** `General`, `#abilities-api`, `7.1`, `dev-notes`, `dev-notes-7-1`, `rest-api`
- **Link:** [https://make.wordpress.org/core/2026/08/05/filtering-registered-abilities-with-wp_get_abilities-in-wordpress-7-1/](https://make.wordpress.org/core/2026/08/05/filtering-registered-abilities-with-wp_get_abilities-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 extends `wp_get_abilities()` to accept an optional `$args` array, enabling built-in filtering of registered abilities by `category`, `namespace`, or `meta` conditions, alongside custom per-item and result-processing callbacks. This replaces the previous pattern where consumers had to manually `array_filter()` the complete registry, consolidating discovery logic into a single shared pipeline. Two new global filters, `wp_get_abilities_item_include` and `wp_get_abilities_result`, allow plugins to influence ability retrieval across the entire site.

## Impact

- **Plugin & theme developers:** Must migrate manual `array_filter()` loops over `wp_get_abilities()` to the new `$args` parameters (`category`, `namespace`, `meta`, `item_include_callback`, `result_callback`) to avoid duplicated filtering logic and inconsistent behavior.
- **REST API consumers:** The `/wp-json/wp-abilities/v1/abilities` endpoint now delegates to `wp_get_abilities()`, exposing `category`, `namespace`, and `meta` as query parameters. Custom metadata fields require explicit schema registration via `rest_abilities_collection_params` to be properly type-coerced from query strings.
- **Backward compatibility:** Existing calls to `wp_get_abilities()` without arguments remain valid, but the function now runs through the global filtering pipeline. Code that previously relied on unfiltered registry retrieval must switch to `WP_Abilities_Registry::get_all_registered()`.
- **No action required** for sites that do not interact with the Abilities API or custom ability discovery.

## Technical details

The change modifies `wp_get_abilities()` to process a declarative filtering pipeline before returning results. The function now accepts an `$args` array supporting:
- `category` (exact string match)
- `namespace` (normalized, delimiter-aware match)
- `meta` (strict key-value matching, supports nested arrays)
- `item_include_callback` (callable receiving `WP_Ability` and `$args`, returning `bool`)
- `result_callback` (callable receiving the matched array, returning transformed array)

The execution order is: declarative matches → `item_include_callback` → `wp_get_abilities_item_include` filter → collection → `result_callback` → `wp_get_abilities_result` filter. The REST controller now uses `wp_get_abilities()` and exposes these filters via query parameters, with boolean annotations (`readonly`, `destructive`, `idempotent`) coerced via `rest_abilities_collection_params`. Raw registry access is explicitly routed to `WP_Abilities_Registry::get_all_registered()` to bypass the new pipeline.

**Before:**
```php
$abilities = array_filter(
    wp_get_abilities(),
    fn( WP_Ability $a ) => 'data-export' === $a->get_category()
);
```

**After:**
```php
$abilities = wp_get_abilities( array( 'category' => 'data-export' ) );
```

## Contribution

Introduced in changeset [62420] for Trac ticket #64990, the feature was developed to eliminate duplicated filtering logic across Core and plugins. Peer review was provided by @benjamin_zekavica, with technical guidance and improvements suggested by @gziolo. The implementation consolidates previously scattered `array_filter()` patterns into a standardized pipeline, with the REST API delegation added to ensure consistent discovery semantics.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
