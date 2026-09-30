# #82692: Site editor v2: Identity takes its form from the view config endpoint

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Enhancement`, `[Package] Edit Site`, `[Feature] Site Editor`
- **Merged:** [`cff8089`](https://github.com/WordPress/gutenberg/commit/cff808931485ebfcc5bf4ffa03b7012e4ee7350e)
- **Discussion:** [#82692](https://github.com/WordPress/gutenberg/pull/82692) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Identity screen in both the extensible site editor (`routes/identity`) and site editor v1 (`packages/edit-site`) now gets its `DataForm` definition from `GET /wp/v2/view-config?kind=root&name=site` instead of a hard-coded `form` constant. The PR adds a base `root`/`site` definition on the PHP side that supplies the form (regular layout, labels on top, fields `title`, `description`, `site_logo`, `site_icon`). The rendered UI is unchanged, but plugins can now alter the form through the `get_entity_view_config_root_site` filter.

## Impact

- **Plugin & theme developers:** A new extension point exists. Filtering `get_entity_view_config_root_site` lets you add, remove, or reorder fields in the Identity form's `form` config. Field definitions (labels, edit components) remain local to the route/screen, so a filter can only reference field ids the JS already defines.
- **Site owners:** No visible change; the screen looks and behaves as before.
- **Headless & REST consumers:** `view-config` now returns a `form` for `kind=root&name=site`. The PR's own test notes that reading root config requires `manage_options`.
- **Gutenberg contributors:** `routes/identity` gains dependencies on `@wordpress/views` and `@wordpress/routes-lock-unlock`.
- The code lives in `lib/compat/wordpress-7.2/` and has a backport changelog entry linked to wordpress-develop PR 13462, so it is targeted at core 7.2 and is tied to the Extensible Site Editor experiment.
- No action required unless you want to customize the form.

## Technical details

**PHP (`lib/compat/wordpress-7.2/view-config-api.php`)**

Adds `_gutenberg_get_entity_view_config_root_site( $data )`, which calls `$data->set()` with a `form` (layout `regular`, `labelPosition` `top`, fields `title`, `description`, `site_logo`, `site_icon`). It is hooked in `gutenberg_register_entity_view_config_filters_7_2()` on `gutenberg_get_entity_view_config_hook_name( 'root', 'site' )` at priority 5. Only `form` is defined; the generic `default_view`, `default_layouts`, and `view_list` are left as built by `gutenberg_get_entity_view_config()`, since the site settings are a singleton that is never listed. Core has no callback for this entity, so this is a base definition rather than a layer on an existing one.

A PHPUnit test, `test_get_items_root_site_form`, asserts the response layout and field list as an admin.

**JS**

- `routes/identity/stage.tsx` and `packages/edit-site/src/components/sidebar-identity/index.jsx` remove the local `form` constant and call `useViewConfig( { kind: 'root', name: 'site', fields: [ 'form' ] } )`. Both return `null` until `form` is available.
- `routes/identity/route.ts` now preloads in the loader via `Promise.all`: the existing `getEntityRecord( 'root', 'site' )` plus `unlock( resolveSelect( coreStore ) ).getViewConfig( 'root', 'site', { fields: 'form' } )`. The comments state the requested fields must match the stage's so both share a cache key.
- `package.json`, `package-lock.json`, and `tsconfig.json` in `routes/identity` gain the `@wordpress/views` and `@wordpress/routes-lock-unlock` references.

Example filter from the PR's test plan, which removes the Site Icon field:

```php
add_filter( 'get_entity_view_config_root_site', function ( $data ) {
    return $data->remove( array( 'form' => array( 'fields' => array( 'site_icon' ) ) ), 1 );
} );
```

## Contribution

Opened and merged as part of the tracking issue Gutenberg #79895, which moves each extensible site editor screen onto the view config endpoint. The PR was authored by @oandregal, and its description says it was implemented with Claude Code and then reviewed and adjusted by the author. The discussion contains only bot comments (bundle size, performance, a flaky e2e test), with no recorded design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
