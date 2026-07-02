# #79686: Icons: Add an icon collections REST endpoint and tighten name rules

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Block] Icon`
- **Merged:** [`a56321a`](https://github.com/WordPress/gutenberg/commit/a56321a410f849235cec25653927bac00d9944df)
- **Discussion:** [#79686](https://github.com/WordPress/gutenberg/pull/79686) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This PR introduces a new REST API endpoint (`/wp/v2/icon-collections`) for registered icon collections and tightens slug validation across the icon system. It also registers `iconCollection` in core-data, enabling client-side scripts to programmatically fetch and enumerate available icon collections alongside the block editor’s icon picker.

## Impact

- **Block & plugin developers**: Can now request available icon collections via `/wp/v2/icon-collections` or use `@wordpress/core-data` (entity: `iconCollection`). No immediate action required for existing implementations, but custom icon registrations must adhere to the new slug pattern.
- **Headless & REST consumers**: The endpoint exposes collection metadata (`slug`, `label`, `description`) with a read-only schema and standard REST authorization checks (`edit_posts` or relevant post type caps).
- **Validation changes**: Icon collection slugs and icon names must now match `/^[a-z0-9]([a-z0-9_-]*[a-z0-9])?$/`. Slugs with leading/trailing hyphens, underscores, or uppercase letters will trigger a `doing_it_wrong` notice and fail registration.

## Technical details

The diff creates `lib/compat/wordpress-7.1/class-wp-rest-icon-collections-controller.php`, implementing `WP_REST_Icon_Collections_Controller` with two `GET` routes under the `wp/v2` namespace. Validation is centralized in `WP_Icon_Collections_Registry::register()` and applied to route parameters in `WP_REST_Icons_Controller_Gutenberg`.

Client-side integration is added in `packages/core-data/src/entities.js`:
```
{
    label: __( 'Icon Collections' ),
    name: 'iconCollection',
    kind: 'root',
    baseURL: '/wp/v2/icon-collections',
    plural: 'iconCollections',
    key: 'slug',
    supportsPagination: false,
},
```
The controller is wired into `rest_api_init` via `gutenberg_register_icon_collections_controller_endpoints()` in `lib/compat/wordpress-7.1/rest-api.php`.

## Contribution

Merged by @t-hamano (with co-authors @mcsf and @mamaduka) as PR #79686. The work followed up on earlier icon registry refactoring (#77260) to ensure collection grouping is fully accessible via the REST layer. Discussion focused on unifying validation patterns between PHP registries and JavaScript core-data, with the strict regex adopted after testing boundary conditions in the PHPUnit suite.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
