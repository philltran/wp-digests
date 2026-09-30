# #82774: Icon block: Don't render non-public icons on the frontend

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`, `[Package] Block library`
- **Merged:** [`a5f8caa`](https://github.com/WordPress/gutenberg/commit/a5f8caac181adfb6063921c2a060f80ea27274ef)
- **Discussion:** [#82774](https://github.com/WordPress/gutenberg/pull/82774) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Icon block's server-side render callback now suppresses output for icons belonging to non-public collections. Previously, `render_block_core_icon()` called `wp_get_icon()` unconditionally, so an icon registered with `public => false` (e.g. `core-admin/wordpress`) would still render in post content even though it is unavailable in the block editor. The fix makes frontend rendering consistent with the editor: non-public icons produce no output.

## Impact

- **Plugin & theme developers:** If you register an icon collection with `public => false` (intended for admin UI only), icons from that collection will no longer render in post content via the Icon block. No code change is required on your side; this is the intended behavior.
- **Site owners / content editors:** Posts that contain an Icon block referencing a non-public icon slug (e.g. `core-admin/wordpress`) will now show an empty block on the frontend instead of the SVG. In practice this is rare because the editor does not offer non-public icons in the inserter.
- **No action required** for the vast majority of sites. No API, hook, or schema change.

## Technical details

The change is in `packages/block-library/src/icon/index.php`, inside `render_block_core_icon( $attributes )`. After the existing early-return for a missing `icon` attribute, a new guard block was inserted:

```php
$registered_icon = WP_Icons_Registry::get_instance()->get_registered_icon( $attributes['icon'] );
if ( null !== $registered_icon ) {
    $icon_collection = WP_Icon_Collections_Registry::get_instance()->get_registered( $registered_icon['collection'] );
    if ( null !== $icon_collection && ! $icon_collection['public'] ) {
        return;
    }
}
```

If the icon resolves to a registered collection whose `public` property is `false`, the function returns `null` (empty output) before reaching the color/rotation/style logic and the `wp_get_icon()` call. Icons in public collections, or icons that are not registered at all (falling through to the existing `wp_get_icon()` path), are unaffected.

A PHPUnit test, `test_renders_only_icons_in_public_collections` in `phpunit/blocks/render-block-icon-test.php`, asserts that `core/caution` (public) produces an `<svg>` element while `core-admin/wordpress` (non-public) produces an empty string, even though `wp_get_icon( 'core-admin/wordpress' )` itself returns non-empty markup.

## Contribution

Opened by @fushar as a follow-up to a design question raised in the review of PR #82634 (the PR that introduced the `public` property on icon collections). @fushar noted the PR depends on a logic change in PR #83261 and must merge after it. Co-authors credited: @tyxla and @t-hamano. No significant design debate is recorded in the three comments; the approach (early-return in the render callback) was accepted as the straightforward fix.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
