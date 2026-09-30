# #82422: Privacy policy page badge: make it visible in Site Editor > Pages and Editor Inspector

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Editor`, `[Package] Fields`
- **Merged:** [`e4ba339`](https://github.com/WordPress/gutenberg/commit/e4ba339cb9e270ad65090838131bd1e3f2d03461)
- **Discussion:** [#82422](https://github.com/WordPress/gutenberg/pull/82422) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg plugin now exposes the privacy policy page (the `wp_page_for_privacy_policy` option) through the REST settings endpoint as `page_for_privacy_policy`. The editor uses it to show a "Privacy Policy Page" badge next to the assigned page in Site Editor > Pages (including the extensible site editor), in the document bar, and in the post card panel. This matches the existing "Homepage" and "Posts Page" badges, which previously had no privacy-policy counterpart in the data views list, even though the classic pages list labels it.

## Impact

**Plugin & theme developers / REST consumers**
- `GET /wp/v2/settings` now returns an integer `page_for_privacy_policy` (default `0`) when the Gutenberg plugin is active and Core doesn't already register the setting with `show_in_rest`. `POST /wp/v2/settings` can also update it.
- Updating it requires `manage_privacy_options`. Without that capability the update is silently ignored: the response is 200 and reports the previous value, rather than returning an error.
- `@wordpress/core-data`'s `Settings` entity type gains `page_for_privacy_policy: number`. TypeScript consumers typing site settings will see the new field.
- The field only exists on the Gutenberg plugin. Code reading it should tolerate `undefined` on Core installs where it hasn't been backported. The backport changelog references wordpress-develop PR 13399 for 7.2.

**Site owners**
- The assigned privacy policy page is now labelled in Site Editor > Pages and in the editor. If the page is also the homepage or posts page, those badges take precedence.

**Hosting & platform**
- No action required.

No breaking changes or deprecations.

## Technical details

**PHP (`lib/compat/wordpress-7.2/rest-api.php`)**
- `gutenberg_register_privacy_policy_page_setting()` is hooked on `rest_api_init` at priority 11, after `register_initial_settings`. It returns early if `get_registered_settings()` already has `wp_page_for_privacy_policy` with a truthy `show_in_rest`. Otherwise it calls `register_setting( 'reading', 'wp_page_for_privacy_policy', ... )` with `show_in_rest => array( 'name' => 'page_for_privacy_policy' )`, `type => 'integer'`, and `default => 0`.
- `gutenberg_restrict_privacy_policy_page_setting_update( $updated, $name )` is hooked on `rest_pre_update_setting`. For `page_for_privacy_policy` it returns `true` (short-circuiting the update) when `! current_user_can( 'manage_privacy_options' )`. This is needed because the settings endpoint only checks `manage_options`, while on multisite `manage_privacy_options` maps to `manage_network`.

**JS**
- `packages/core-data/src/entity-types/settings.ts`: adds `page_for_privacy_policy: number` to `Settings`.
- `packages/editor/src/utils/pageTypeBadge.js`: `usePageTypeBadge` also computes `isPrivacyPolicyPage` from `siteSettings?.page_for_privacy_policy === _postId`. It returns `__( 'Privacy Policy Page' )` after the front-page and posts-page checks. This drives the document bar and post card panel.
- `packages/fields/src/fields/page-title/view.tsx`: `PageTitleView` reads `page_for_privacy_policy` and replaces the `[ frontPageId, postsPageId ].includes(...)` check with an if/else chain (Homepage, then Posts Page, then Privacy Policy Page) that renders a single `WCBadge`.

**Tests**
- New `phpunit/privacy-policy-page-setting-test.php` covers registration, GET, authorized update, and ignored update when `manage_privacy_options` is denied via `map_meta_cap`.

```php
// Before: no page_for_privacy_policy in /wp/v2/settings
// After (Gutenberg plugin): { ..., "page_for_privacy_policy": 42 }
```

## Contribution

The PR closes #67517 and was written by @oandregal with Claude Code assistance, with the author reporting manual review and testing. @jasmussen approved the design and asked whether long titles and the badge wrap in the inspector; the author showed that they do, as for existing badges. Review was also requested from @Mamaduka and @ntsekouras, and the author asked them on the thread whether anything was left to address. A CodeRabbit note about Markdown lint in the backport changelog was left open at the last review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
