# #78977: Extract entity view config into a filterable API

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Feature] Extensibility`, `[Type] New API`, `[Feature] DataViews`
- **Merged:** [`84ccee8`](https://github.com/WordPress/gutenberg/commit/84ccee886a9450080d8895ff1ef2464f4934eaf0)
- **Discussion:** [#78977](https://github.com/WordPress/gutenberg/pull/78977) · 2 comments · 0 reactions

## Summary

The DataViews configuration previously hardcoded inside the `Gutenberg_REST_View_Config_Controller_7_1` REST controller has been extracted into a standalone, filterable API. This change decouples the data model from the REST transport layer and introduces `gutenberg_get_entity_view_config()`, allowing third-party plugins and theme developers to override view settings per entity kind/name without modifying core files. The `/wp/v2/view-config` endpoint behavior and response shape remain completely unchanged.

## Impact

- **Plugin & Theme Developers**: Can now extend or modify DataViews configuration for custom entities by attaching callbacks to the dynamic `get_entity_view_config_{$kind}_{$name}` filter.
- **Hosting & Platform Teams**: No action required. The `/wp/v2/view-config` endpoint returns identical JSON payloads as before.
- **Headless & REST Consumers**: No action required. Schema and response structure are unchanged.
- **Breaking Changes / Deprecations**: None. All existing controller methods were moved, not altered in logic.

## Technical details

- A new file `lib/compat/wordpress-7.1/view-config-api.php` introduces the public function `gutenberg_get_entity_view_config( string $kind, string $name ): array`.
- The function constructs a shared default configuration (`default_view`, `default_layouts`, `view_list`, `form`) and applies the dynamic filter `get_entity_view_config_{$kind}_{$name}` to allow overrides before returning the final array.
- Inside `Gutenberg_REST_View_Config_Controller_7_1::get_items()`, all private methods handling entity-specific logic (e.g., `get_default_view_for_page()`, `get_view_list_for_wp_block()`) were removed. The controller now delegates to the new API:
  ```php
  // Before
  if ( 'postType' === $kind && 'page' === $name ) {
      $default_view = $this->get_default_view_for_page();
      // ...
  }

  // After
  $config = gutenberg_get_entity_view_config( $kind, $name );
  ```
- The controller reads directly from the returned `$config` array to populate the response payload. No database changes, REST route modifications, or schema alterations were introduced.

## Contribution

Opened by @oandregal as part of ongoing DataViews extensibility work, following up on core issues #76544 and PR #76573. Co-authored by @ntsekouras and merged in commit `84ccee8`. The PR focused strictly on extracting hardcoded view logic into a filterable API to decouple the model from the transport layer; no alternative implementations or rejected approaches were detailed in the provided discussion. The change was validated against existing DataViews screens and the `/wp/v2/view-config` endpoint to ensure byte-for-byte parity with trunk.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
