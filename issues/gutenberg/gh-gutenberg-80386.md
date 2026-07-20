# #80386: Fix default aspect ratio for lazy loaded Featured image

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Post Featured Image`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`3a543ef`](https://github.com/WordPress/gutenberg/commit/3a543ef4eb136ccf16ce250ad9106660ec752c6b)
- **Discussion:** [#80386](https://github.com/WordPress/gutenberg/pull/80386) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Featured Image block now correctly omits the `aspect-ratio` CSS property when its attribute value is `"auto"`, preventing layout shifts and broken sizing on lazy-loaded images. Previously, legacy blocks or hand-written markup containing `"aspectRatio":"auto"` would output an ineffective CSS rule, causing images to render incorrectly. This fix ensures the block falls back to the image's intrinsic dimensions when no explicit aspect ratio is set.

## Impact

- **Theme & plugin developers:** No API changes or deprecations. Existing block usage remains compatible.
- **Site owners & editors:** Fixes visual sizing issues for posts/pages saved with older Gutenberg versions where the aspect ratio was set to "Original" (which historically saved as `"auto"`).
- **No action required:** The fix is backward-compatible and automatically applies to existing blocks on render or save.

## Technical details

The change modifies `packages/block-library/src/post-featured-image/edit.js` and `packages/block-library/src/post-featured-image/index.php`. In the JS editor component, a new variable `aspectRatioStyle` is computed as `aspectRatio === 'auto' ? undefined : aspectRatio`, which is then passed to the `style` prop for both the placeholder and the rendered image. In the PHP renderer, the `render_block_core_post_featured_image` function now conditionally appends the CSS property:

```php
// Before
$extra_styles .= esc_attr( safecss_filter_attr( 'aspect-ratio:' . $attributes['aspectRatio'] ) ) . ';';

// After
if ( 'auto' !== $attributes['aspectRatio'] ) {
    $extra_styles .= esc_attr( safecss_filter_attr( 'aspect-ratio:' . $attributes['aspectRatio'] ) ) . ';';
}
```

This prevents `aspect-ratio: auto` from being output in the inline CSS, allowing the browser to calculate the ratio from the image's natural dimensions.

## Contribution

Opened and merged by @tellthemachines, with review and validation from @ramonjd, @andrewserong, and @tommusrhodus. The discussion clarified that `"aspectRatio":"auto"` only appears in legacy content saved by Gutenberg versions prior to 23.5 (when the UI used a `SelectControl` that wrote the value verbatim). The fix was cherry-picked to `wp/7.1` and backported to the Gutenberg RC.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
