# #77290: Fix Gutenberg_REST_View_Config_Controller_7_1 PHP warnings

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Feature] DataViews`
- **Merged:** [`6d4534e`](https://github.com/WordPress/gutenberg/commit/6d4534e1aca9832fc4663c5b8b71fbadde16f848)
- **Discussion:** [#77290](https://github.com/WordPress/gutenberg/pull/77290) · 3 comments · 0 reactions

## Summary

Resolves a PHP `Undefined array key "layout"` warning in `Gutenberg_REST_View_Config_Controller_7_1` when fetching view configuration for `wp_template_part` templates. The update populates the missing `layout` key within the default grid structure, ensuring the REST endpoint returns a fully formed data contract without triggering notices in strict error-reporting environments.

## Impact

- **Theme & plugin developers** querying view configs via `Gutenberg_REST_View_Config_Controller_7_1` will receive stable arrays that match expected nested structures, eliminating intermittent PHP warnings.
- **Hosting & platform teams** operating with `WP_DEBUG` or strict error reporting will see suppressed notices without requiring configuration overrides.
- No migration steps or code changes required; this is a transparent stability fix for existing view config consumers.

## Technical details

The change modifies `lib/compat/wordpress-7.1/class-gutenberg-rest-view-config-controller-7-1.php` inside the private method `get_default_layouts_for_wp_template_part()`. The diff replaces an empty grid fallback:
```diff
-					'grid'   => array(),
+					'grid'   => array(
+						'layout' => array(),
+					),
```
This injects the missing `layout` key, aligning the serialized default layouts with consumer expectations that anticipate a nested structure under grid templates. The fix directly addresses the specific schema gap identified in the follow-up to #76734 without altering REST route behavior or block registration logic.

## Contribution

Opened by @ntsekouras as a targeted follow-up to #76734 to close a structural gap in the initial view config serialization. Reviewed and merged at commit 6d4534e, with @oandregal coordinating the backport to the corresponding WordPress 7.1 core PR. The patch focuses exclusively on populating the missing `layout` array key rather than refactoring the broader data flow.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
