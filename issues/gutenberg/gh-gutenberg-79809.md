# #79809: View Config: Add version handling

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`
- **Merged:** [`977d675`](https://github.com/WordPress/gutenberg/commit/977d6754af5cc1c81738e6fc9b1fc707b676cc30)
- **Discussion:** [#79809](https://github.com/WordPress/gutenberg/pull/79809) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Introduces versioned patching for the server-side view configuration API used by DataViews-powered screens (e.g., Pages list, Quick Edit). The `get_entity_view_config_{$kind}_{$name}` filter now receives a `Gutenberg_View_Config_Data` container instead of a raw array, providing methods like `update_properties()`, `update_view_list_items()`, and `update_form_fields()` that apply version-tagged patches. This prevents imperative array manipulation from breaking on future schema changes and ensures patches compose safely across core and plugins.

## Impact

- **Plugin & theme developers:** The `get_entity_view_config_{$kind}_{$name}` filter signature changed: callbacks must now receive an object, call methods on it, and return it instead of mutating a raw array.
- **Breaking/Adoption required:** Any existing code that manually walks or unsets keys in the raw configuration will break. Developers must migrate to the new container methods.
- **Headless & REST consumers:** The `GET /wp/v2/view-config` response now includes a readonly `version` integer field indicating the configuration schema version exposed in the OpenAPI schema.

## Technical details

- Introduces `lib/compat/wordpress-7.1/class-gutenberg-view-config-data.php` containing `class Gutenberg_View_Config_Data`.
- The container exposes targeted patching methods, all requiring a `$version` argument (currently hardcoded to `1`):
  - `update_properties( $patch, $version )`: Merges into `default_view`, `default_layouts`, and non-field `form` properties. Passes `null` for a key resets it to default.
  - `update_view_list_items( $items, $version )`: Patches the `view_list` array keyed by `slug`. Matches merge in place, unknown slugs append, `null` removes.
  - `update_form_fields( $fields, $version )`: Patches `form` fields keyed by `id`, traversing nested `children`. Same merge/remove rules apply.
  - `set( $key, $value, $version )`: Replaces a whole top-level key, halting inheritance of core changes for that key.
- The REST controller (`Gutenberg_REST_View_Config_Controller_7_1`) now injects `'version' => Gutenberg_View_Config_Data::LATEST_VERSION` into the response payload and exposes it in the OpenAPI schema as an integer `version` field.

## Contribution

Opened by @ntsekouras with co-authorship from @mcsf and @Mamaduka, this PR shifts the DataViews config filter payload from a mutable array to a versioned container modeled after `WP_Theme_JSON_Data->update_with()`. Review feedback questioned the necessity of a generic cross-entity filter and how private methods would be extensible; the author clarified that dedicated per-entity filters are preferable and that the class will remain indefinitely in the Gutenberg plugin. The merged diff finalized the full data class, updated the REST schema to expose `version`, and added comprehensive documentation to `filters-and-hooks.md`. Merged at commit `977d675`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
