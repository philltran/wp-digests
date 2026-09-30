# #81889: Try caching gutenberg_get_global_styles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Performance`, `Global Styles`
- **Merged:** [`7f2be02`](https://github.com/WordPress/gutenberg/commit/7f2be02a0bdeed98a2e1da8de135233207d08f25)
- **Discussion:** [#81889](https://github.com/WordPress/gutenberg/pull/81889) · 7 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

`gutenberg_get_global_styles()` now caches its result in the `theme_json` object-cache group, using the same approach `gutenberg_get_global_settings()` already uses. Previously every call re-ran `WP_Theme_JSON_Resolver_Gutenberg::get_merged_data()` (and optionally `resolve_variables()`), which is why the layout block support kept a function-level `static` copy. The static is removed and the cache is invalidated by the existing theme-JSON cache-clearing routine, including on theme switch.

## Impact

- **Site owners:** No visible change; this is a performance improvement for pages rendering many layout blocks, since global styles are no longer recomputed on each call.
- **Plugin & theme developers:** No API changes. Code calling `gutenberg_get_global_styles()` gets the cached array, so it benefits automatically. Code that changes theme.json data programmatically outside the paths that trigger `_gutenberg_clean_theme_json_caches()` may see stale values when `WP_DEBUG` is off.
- **Hosting & platform:** With a persistent object cache, four new keys appear in the `theme_json` group (per origin, plain and `_resolved`). Without one, `wp_cache_*` is per-request, so the effect is limited to deduplicating calls within a request.
- **Core backport:** A `backport-changelog/7.2/13214.md` entry links this to `wordpress-develop` PR 13214, so the change is targeted at WordPress 7.2.
- No action required.

## Technical details

**`lib/global-styles-and-settings.php`**

`gutenberg_get_global_styles()` now builds a cache key of `'gutenberg_get_global_styles_' . $origin`, with `_resolved` appended when `$context['transforms']` contains `resolve-variables`. It reads via `wp_cache_get( $cache_key, 'theme_json' )`. On a miss (`false`) or when `WP_DEBUG` is true, it computes merged data, applies `WP_Theme_JSON_Gutenberg::resolve_variables()` if requested, takes `get_raw_data()['styles']`, and calls `wp_cache_set()`. The `$path` lookup via `_wp_array_get()` is applied to the cached array afterward, so the cache stores the full styles tree rather than per-path results.

`_gutenberg_clean_theme_json_caches()` now also deletes:

- `gutenberg_get_global_styles_custom`
- `gutenberg_get_global_styles_custom_resolved`
- `gutenberg_get_global_styles_theme`
- `gutenberg_get_global_styles_theme_resolved`

These match the keys generated for the `custom` and `theme` origins. Other origins that could generate keys, such as the default `custom`-style path for other values, are not explicitly cleared in the diff. The docblock is updated to say the function cleans caches used by the theme JSON APIs.

**`lib/block-supports/layout.php`**

The `static $global_styles = null;` in `gutenberg_render_layout_support_flag()` and its lazy-init check are removed. The function now calls `gutenberg_get_global_styles()` directly each time it needs the default `blockGap`, relying on the new cache. Per the PR description, the old static did not refresh on theme switch, which caused the PHP test failures in #68002.

```php
// Before
if ( null === $global_styles ) {
	$global_styles = gutenberg_get_global_styles();
}
// After
$global_styles = gutenberg_get_global_styles();
```

## Contribution

Opened by @tellthemachines as an alternative to #81852, aimed at fixing PHP test failures in #68002 caused by the stale static not resetting when tests switched themes. It was initially pointed at the #68002 branch to verify CI, which unintentionally pinged several people, then rebased onto trunk. @ramonjd reviewed it, confirmed the cache keys match those cleared, and manually tested a layout-heavy page across a theme switch in wp-env with `WP_DEBUG` off. Some e2e tests kept failing; both authors suspected they were unrelated to this PR, since the failures also appeared on the List-block branch it was derived from and could not be reproduced locally. The PR discloses AI assistance (codex).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
