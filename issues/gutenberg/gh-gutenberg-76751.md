# #76751: Revert client-side media processing plugin-only gate

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Status] In Progress`, `[Type] Feature`, `No Core Sync Required`, `[Feature] Client Side Media`
- **Merged:** [`0a801ac`](https://github.com/WordPress/gutenberg/commit/0a801acfea19af0495988a8eef72a4bc1223474e)
- **Discussion:** [#76751](https://github.com/WordPress/gutenberg/pull/76751) · 7 comments · 1 reactions

## Summary

PR #76751 reverts #76700, which had placed client-side media processing (libvips/WebAssembly image processing in the browser) behind an `IS_GUTENBERG_PLUGIN` gate. With WordPress 7.1 now open for commits, the feature is restored to both the Gutenberg plugin and core code paths. Two guards are removed: the conditional wrapper around the VIPS loader action registration in `lib/client-assets.php`, and the early-return in `lib/media/load.php` that short-circuited the entire feature for non-plugin contexts. The feature remains gated by the existing `gutenberg_is_client_side_media_processing_enabled()` check.

## Impact

**Plugin & theme developers / block editor integrators**
- The `@wordpress/vips/loader` script module is now enqueued via `enqueue_block_editor_assets` unconditionally — no longer contingent on `IS_GUTENBERG_PLUGIN` being defined and truthy. Any tooling that suppressed or tested for this module only in Gutenberg plugin contexts must be updated.
- No deprecated symbols or removed filters; this is a removal of a guard, not an API change.

**Hosting & platform**
- Client-side media processing (WASM/libvips) will be active in vanilla WordPress core whenever `gutenberg_is_client_side_media_processing_enabled()` returns `true`. Environments that had assumed this was a plugin-only concern should audit upload pipeline behavior and server-side `image_output_formats` REST index exposure.
- A known post-merge test regression exists: `Media_Processing_Test::test_get_rest_index_should_return_additional_settings_can_upload_files` fails because `image_output_formats` appears in the REST index contrary to test expectations. A fix is tracked in PR #78788.

**No action required** for sites not customizing the media upload pipeline.

## Technical details

**`lib/client-assets.php` (~line 441)**

Before:
```php
if ( defined( 'IS_GUTENBERG_PLUGIN' ) && IS_GUTENBERG_PLUGIN ) {
    add_action( 'enqueue_block_editor_assets', 'gutenberg_enqueue_vips_loader' );
}
function gutenberg_enqueue_vips_loader() {
    wp_enqueue_script_module( '@wordpress/vips/loader' );
}
```

After:
```php
add_action( 'enqueue_block_editor_assets', 'gutenberg_enqueue_vips_loader' );
function gutenberg_enqueue_vips_loader() {
    wp_enqueue_script_module( '@wordpress/vips/loader' );
}
```

The `add_action` call is now unconditional. `gutenberg_enqueue_vips_loader()` calls `wp_enqueue_script_module( '@wordpress/vips/loader' )`, loading the WASM-backed VIPS image processing module into the block editor.

**`lib/media/load.php` (~line 15)**

Before:
```php
// Client-side media processing is currently plugin-only while the feature matures.
if ( ! defined( 'IS_GUTENBERG_PLUGIN' ) || ! IS_GUTENBERG_PLUGIN ) {
    return;
}

if ( ! gutenberg_is_client_side_media_processing_enabled() ) {
    return;
}
```

After:
```php
if ( ! gutenberg_is_client_side_media_processing_enabled() ) {
    return;
}
```

The plugin-only early-return is gone. The capability check via `gutenberg_is_client_side_media_processing_enabled()` remains the sole gate. The REST index exposure of `image_output_formats` — which surfaced a test failure — originates downstream of this file now that the early-return no longer prevents execution in core.

## Contribution

@adamsilverstein authored both the original gate (PR #76700, commit `d259aab6b37`) and this revert. The PR was held until the WordPress 7.1 development window opened (announced 2026-05-20) and the corresponding wordpress-develop PR #11324 landed. @swissspidy is credited as co-author. After merge at `0a801ac`, @alecgeatches identified the `Media_Processing_Test` REST index regression; @adamsilverstein acknowledged it and is addressing it in PR #78788.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
