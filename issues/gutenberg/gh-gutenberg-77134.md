# #77134: Add `date` field in templates and template parts

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] Templates API`, `REST API Interaction`
- **Merged:** [`971a4c0`](https://github.com/WordPress/gutenberg/commit/971a4c02270fa7ba4d0be197c32f86c0cd2a1872)
- **Discussion:** [#77134](https://github.com/WordPress/gutenberg/pull/77134) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Templates API now exposes a `date` field alongside the existing `modified` field for `wp_template` and `wp_template_part` post types via the REST API. This addition standardizes how creation timestamps are accessed, simplifying revision handling and aligning template metadata with standard post object behavior.

## Impact

- **Plugin & theme developers**: REST API consumers querying `/wp/v2/templates` or `/wp/v2/template-parts` will now receive a `date` property (ISO 8601 string or `null`). No immediate code changes are required, but clients relying on `modified` for creation timestamps should migrate to `date`.
- **Headless & REST consumers**: Schema validation will now expect `date` as a string or null. Responses that previously omitted it will include it.
- **No action required** for existing implementations unless they explicitly parse or filter template REST responses.

## Technical details

The change registers a new REST field via `register_rest_field()` in `lib/compat/wordpress-7.1/rest-api.php`. It targets both `wp_template` and `wp_template_part` post types. The `get_callback` resolves the underlying post ID from `$item['wp_id']`, fetches the post object, and returns `mysql_to_rfc3339( $post->post_date )` if `post_date` is set, otherwise `null`. The schema defines `date` as `['string', 'null']` with `format: 'date-time'`, contexts `['view', 'edit']`, and `readonly: true`. A review noted that `mysql_to_rfc3339()` can return `false` on empty input, so the callback explicitly guards against `null`/empty values to maintain schema compliance. Unit tests verify the field appears in `get_item`, `get_items`, and schema inspection.

## Contribution

Opened by @ntsekouras and merged by @mamaduka (co-authored), this PR was extracted from a larger templates API overhaul (#77086). During review, @tyxla flagged a potential type mismatch where `mysql_to_rfc3339()` could return `false` for file-backed templates lacking a date, prompting a guard in the callback to ensure strict `string|null` compliance. The change shipped with dedicated PHPUnit tests and a backport changelog entry for WordPress 7.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
