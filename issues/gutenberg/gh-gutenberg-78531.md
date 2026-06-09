# #78531: Fix registered post meta scope

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Bug`
- **Merged:** [`9d0dc3c`](https://github.com/WordPress/gutenberg/commit/9d0dc3c2df0e61920a7690ba9418626713cdaa99)
- **Discussion:** [#78531](https://github.com/WordPress/gutenberg/pull/78531) · 3 comments · 0 reactions

## Summary

Fixes a bug where a registered post meta schema for editor CRDT/collaboration state was exposed across all REST API contexts. The change restricts the meta's visibility to the `edit` context only, preventing editor-internal state data from leaking into standard `view` responses and aligning the schema with WP's default REST context scoping rules.

## Impact

- **Headless & REST consumers:** Post meta registered by the collaboration compatibility layer will no longer appear in standard `view` context responses. Only requests explicitly asking for `context=edit` (typically internal editor payloads) will return this data.
- **Plugin & theme developers:** No code changes required for most use cases. If you fetch or filter this specific post meta via custom REST endpoints using a `view` context, expect it to now return empty/null and adjust client-side fallbacks accordingly.
- **Hosting & platform teams:** No configuration changes needed; this is a targeted schema scoping fix.

## Technical details

- **File change:** `lib/compat/wordpress-7.1/collaboration.php` (renamed from `lib/compat/wordpress-7.0/collaboration.php`). Other collaboration files were similarly moved to the `wordpress-7.1` directory.
- **Function affected:** `gutenberg_rest_api_crdt_post_meta()`
- **Schema update:** The `'show_in_rest'` registration was changed from a bare boolean `true` to an explicit array defining the schema and restricting context:
  ```php
  // Before
  'show_in_rest' => true,

  // After
  'show_in_rest' => array(
      'schema' => array(
          'type'    => 'string',
          'context' => array( 'edit' ),
      ),
  ),
  ```
- **Behavioral note:** By default, `register_post_meta()` exposes data to all contexts when `show_in_rest` is `true`. Explicitly setting `'context' => array( 'edit' )` enforces strict separation between editor state and public-facing REST responses.

## Contribution

Opened and merged by @alecgeatches (co-authored by @peterwilsoncc) as a bug fix resolving #78532. The diff shows a straightforward schema scoping correction alongside a compatibility directory rename to align with WordPress 7.1 build targets. Maintainer @t-hamano confirmed the fix was shipped in Gutenberg plugin v23.2 and determined no backport to older minor releases was necessary.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
