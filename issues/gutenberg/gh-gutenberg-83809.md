# #83809: Template Part: Fall back to rendering a template part from the theme file if the db version is unavailable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Template Part`
- **Merged:** [`dbe580a`](https://github.com/WordPress/gutenberg/commit/dbe580aee30f5ad24736003d813302bbfe8c4100)
- **Discussion:** [#83809](https://github.com/WordPress/gutenberg/pull/83809) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Template Part block's server-side render (`render_block_core_template_part()`) now checks whether `_build_block_template_result_from_post()` returned a `WP_Error` before reading `->content`. If the database-saved (customized) template part can't be built, it reports the error via `wp_trigger_error()` and falls back to the theme's template part file, rather than rendering nothing and emitting an `Undefined property: WP_Error::$content` warning.

## Impact

- **Site owners / hosts using persistent object cache:** Customized template parts (e.g. header/footer) should no longer disappear from the frontend if `get_the_terms()` transiently returns no `wp_theme` terms for the post. The theme file version renders instead, so the site shows the original rather than a blank region.
- **Plugin & theme developers:** No API changes. Behavior change: in this failure state you may now see a `wp_trigger_error()` notice (`Error when loading the template part … from the database: …`) in logs where previously there was a PHP warning about an undefined property. If you filter `get_the_terms` for `wp_theme` and return a non-array value, the block now falls back instead of erroring.
- **Headless / REST consumers:** Not affected.
- No action required.

## Technical details

Change in `packages/block-library/src/template-part/index.php`, in the branch that handles a template part looked up from the database via `WP_Query` on `wp_template_part`:

**Before:** if `$template_part_query->have_posts()` yielded a post, the code called `_build_block_template_result_from_post()` and immediately read `$block_template->content` (and `->area`). `_build_block_template_result_from_post()` can return a `WP_Error` (e.g. when the post's `wp_theme` terms can't be read, even though the query matched on them), so `->content` was an undefined property.

**After:**

```php
$block_template = $template_part_post ? _build_block_template_result_from_post( $template_part_post ) : null;
if ( is_wp_error( $block_template ) ) {
	wp_trigger_error( __FUNCTION__, sprintf( /* translators */ __( 'Error when loading the template part %1$s from the database: %2$s' ), $template_part_id, $block_template->get_error_message() ) );
}
if ( $block_template && ! is_wp_error( $block_template ) ) {
	$content = $block_template->content;
	// area handling unchanged
}
```

When the result is a `WP_Error`, `$content` stays unset, so the existing code path that loads from the theme file runs (`render_block_core_template_part_file` action), mirroring how core's `get_block_template()` falls back to the file on error. If there's also no theme file, the block renders the existing "deleted or unavailable" output and fires `render_block_core_template_part_none`.

A new PHPUnit file, `phpunit/blocks/render-template-part-test.php`, covers: normal customized render, an emptied `wp_theme_relationships` cache, `get_the_terms` filtered to `false`, `get_the_terms` returning a `WP_Error`, and the no-theme-file case. It captures errors through the `wp_trigger_error_always_run` action and disables them with the `wp_trigger_error_trigger_error` filter. The `packages/block-library/CHANGELOG.md` also gets an entry.

## Contribution

Opened by @andrewserong as a defensive follow-up to an earlier guard for failures when fetching the template part from file (#69309). The root cause isn't fully understood: the author suspects a race condition with object caching in hosted environments where `get_the_terms()` returns no `wp_theme` terms. The PR description says Claude Code was used heavily for the investigation and tests. Review was brief, with @tyxla approving and props credited to @talldan.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
