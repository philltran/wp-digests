# #83762: Extensible Site Editor: Pass custom block categories to the blocks store

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`
- **Merged:** [`7ba4164`](https://github.com/WordPress/gutenberg/commit/7ba41647202235b3f90e2ffb108ee13ec6bea1e3)
- **Discussion:** [#83762](https://github.com/WordPress/gutenberg/pull/83762) · 2 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

In the Extensible Site Editor experiment, the settings endpoint returned `blockCategories` but nothing in the asset pipeline called `wp.blocks.setCategories()` with them, so blocks registered with a custom category lost it and appeared under "Uncategorized" in the inserter. This PR injects a `wp.blocks.setCategories()` inline script into the assets endpoint, placed before server block definitions, so categories are known before any `registerBlockType()` call drops an unrecognized category.

## Impact

- **Plugin & theme developers (Extensible Site Editor experiment only):** Custom block categories registered via the `block_categories_all` filter now appear correctly in the site editor inserter. No code changes required on your side; the fix is entirely in the experiment's asset pipeline.
- **All other editors (post editor, site editor with experiment off):** No change. The fix is scoped to the `get_assets` method in the experimental controller.
- **No action required** unless you are actively testing the Extensible Site Editor experiment and were seeing custom-category blocks fall into "Uncategorized".

## Technical details

The change is in `lib/experimental/class-wp-rest-block-editor-settings-controller.php`, inside the `get_assets()` method. Seven lines are added before the existing "Preload blocks" section:

```php
// Before block registration: `registerBlockType()` drops an unknown `category`.
wp_add_inline_script(
    'wp-block-library',
    'wp.blocks.setCategories(' . wp_json_encode( get_block_categories( new WP_Block_Editor_Context() ), JSON_HEX_TAG | JSON_UNESCAPED_SLASHES ) . ');',
    'before'
);
```

The `'before'` position ensures the `setCategories` call executes before the inline scripts that call `registerBlockType()` for server-side blocks. Without this, `registerBlockType()` silently discards a `category` value it does not recognize, and the block lands in the default "Uncategorized" group in the inserter. The categories are fetched via `get_block_categories( new WP_Block_Editor_Context() )`, which applies the `block_categories_all` filter in the appropriate context.

## Contribution

Opened by @ntsekouras to fix issue #83759. The merge commit credits @youknowriad as co-author, suggesting review or direction from that contributor. The PR notes use of "Fable 5.1 with direction and review" for AI-assisted authoring. The discussion record contains only automated performance and flaky-test reports; no design debate or alternative approaches are visible.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
