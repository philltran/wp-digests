# #78790: Fix type of `$block_instance` parameter in `block_core_image_render_lightbox()`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @westonruter
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Gallery`
- **Merged:** [`08c646c`](https://github.com/WordPress/gutenberg/commit/08c646ced0f13f05cda975a8bd03ebf6690c92fc)
- **Discussion:** [#78790](https://github.com/WordPress/gutenberg/pull/78790) · 3 comments · 0 reactions

## Summary

The `$block_instance` parameter in `block_core_image_render_lightbox()` was incorrectly declared as `array` in both the PHPDoc and the function signature. It is actually a `WP_Block` object, as is standard for the third argument passed to `render_block` filter callbacks. This corrects the type hint added in [#62906](https://github.com/WordPress/gutenberg/pull/62906) / commit `1077e73`.

## Impact

**Plugin & theme developers**
- If you have code that calls `block_core_image_render_lightbox()` directly (uncommon but possible in test or compatibility shims), update any `array` type hints on the third argument to `WP_Block`.
- If you hooked into this function indirectly or wrote PHPStan/Psalm stubs based on the old signature, update those stubs.
- No action required for the vast majority of sites — the runtime behavior of the Image block lightbox is unchanged; only the declared type is corrected.

## Technical details

The change is entirely in `packages/block-library/src/image/index.php`.

**Before:**
```php
/**
 * @param string $block_content  Rendered block content.
 * @param array  $block          Block object.
 * @param array  $block_instance Block instance.
 */
function block_core_image_render_lightbox( $block_content, $block, $block_instance ) {
```

**After:**
```php
/**
 * @param string   $block_content  Rendered block content.
 * @param array    $block          Block object.
 * @param WP_Block $block_instance Block instance.
 */
function block_core_image_render_lightbox( $block_content, array $block, WP_Block $block_instance ) {
```

The fix adds a native PHP `WP_Block` type declaration on `$block_instance` (and simultaneously adds `array` on `$block`, which previously had no native hint). Both changes align the signature with what WordPress's block rendering pipeline actually passes as the third argument to `render_block` filter callbacks. No logic inside the function body was modified.

## Contribution

Opened and authored by **@westonruter** as a direct follow-up to PR #62906 (merged earlier). **@t-hamano** was credited as a co-author in the merge commit. The PR was merged as commit `08c646c`. Discussion was minimal — three bot comments covering label enforcement and contributor attribution; no design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
