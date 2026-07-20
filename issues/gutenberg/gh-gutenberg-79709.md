# #79709: Guidelines: Add Blocks as a registry scope

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @gziolo
- **Labels:** `[Type] Enhancement`, `[Feature] Extensibility`, `[Feature] Guidelines`
- **Merged:** [`ce0129a`](https://github.com/WordPress/gutenberg/commit/ce0129a1f77baa71ae5190ff44dba190ac954611)
- **Discussion:** [#79709](https://github.com/WordPress/gutenberg/pull/79709) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Guidelines feature now registers the `blocks` scope in the `wp_guideline_scopes()` registry, moving it from a hardcoded client-side entry to a server-managed scope. This change allows plugins and themes to add, reorder, or remove the Blocks section via the `wp_guideline_scopes` filter, making it consistent with other guideline sections. It also replaces the `GUTENBERG_GUIDELINE_MAX_LENGTH` constant with a filterable `wp_guideline_max_length()` function and tightens the REST guard to strictly validate guideline row slugs against registered scopes.

## Impact

- **Plugin & theme developers:** Can now remove or customize the Blocks guideline section using the `wp_guideline_scopes` filter. The `GUTENBERG_GUIDELINE_MAX_LENGTH` constant is removed; use the `wp_guideline_max_length` filter instead if you previously relied on it.
- **Site owners:** No action required. The Settings → Guidelines UI behaves identically, but the Blocks section is now fully controllable via code.
- **Headless & REST consumers:** The REST guard for `wp_knowledge` rows now strictly validates slugs against registered scopes. Unrecognized `guideline-` prefixed slugs are no longer sanitized or capped, which may affect custom integrations relying on the previous broad prefix matching.

## Technical details

- **Registry & Scope Resolution:** `wp_guideline_scopes()` in `lib/experimental/knowledge/knowledge.php` now includes a `blocks` entry (order 40). The `wp_guideline_scope_from_slug()` helper was rewritten to prioritize exact scope matches over the `guideline-block-` prefix, ensuring `guideline-block-*` rows resolve to `blocks` only when the scope is registered.
- **REST Guard & Sanitization:** `wp_knowledge_guard_guideline_row()` now calls `wp_guideline_scope_from_slug()` to determine if a row should be shaped. It skips title localization for the `blocks` scope (preserving per-block canonical names) and uses `wp_guideline_max_length()` for content capping. Rows that resolve to `null` are left untouched.
- **Constant to Filter Migration:** The `GUTENBERG_GUIDELINE_MAX_LENGTH` constant was removed and replaced with `wp_guideline_max_length()`, which applies the `wp_guideline_max_length` filter (default 5000).
- **Client-side Alignment:** `routes/guidelines/data.ts` and `import-export.ts` now check for the `blocks` scope in the registry before querying or importing per-block rows, preventing orphaned queries when the scope is filtered out.
- **Before/After (Filter usage):**
  ```php
  // Before: Blocks section was hardcoded on the client and couldn't be removed.
  // After: Plugins can now remove it via the registry filter.
  add_filter( 'wp_guideline_scopes', function ( $scopes ) {
      unset( $scopes['blocks'] );
      return $scopes;
  } );
  ```

## Contribution

Opened and merged by @gziolo, with co-authorship from @jorgefilipecosta. The PR aligns the Gutenberg implementation with parallel work in WordPress Core (Trac #65476). Review focused on ensuring the `blocks` scope behaves as a multi-row container while remaining filterable, and on tightening the REST guard to prevent orphaned rows. The larger type-assignment rework was explicitly deferred to a follow-up PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
