# #79346: Block Bindings: Preserve nested lists when binding List Item content

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @cbravobernal
- **Labels:** `[Type] Bug`
- **Merged:** [`a02c4d0`](https://github.com/WordPress/gutenberg/commit/a02c4d03567b3ed27eab0824a83769ed2903e452)
- **Discussion:** [#79346](https://github.com/WordPress/gutenberg/pull/79346) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

When binding the `content` attribute of a `core/list-item` block, WordPress versions prior to 7.1 would drop any nested inner blocks because `WP_Block::replace_html()` replaced the entire `<li>` element. This PR adds a compatibility workaround in the Gutenberg plugin that detects when a list item's content is bound and re-appends the rendered inner blocks before the closing `</li>` tag. This ensures nested lists and other inner blocks remain intact when using block bindings on older WordPress versions, while acting as a no-op on 7.1+ where Core already handles this correctly.

## Impact

- **Plugin & theme developers:** If you use block bindings to replace `core/list-item` content on sites running WordPress < 7.1, nested inner blocks (lists, buttons, etc.) will now render correctly instead of being stripped.
- **Hosting & platform teams:** No configuration changes required. The fix is bundled in the Gutenberg plugin and automatically applies when the plugin is active on WP < 7.1.
- **No action required** for sites on WordPress 7.1 or later, as Core's `WP_Block::replace_html()` already preserves inner blocks. The workaround detects this and skips execution.

## Technical details

The change introduces `lib/compat/wordpress-7.1/block-bindings.php` in the Gutenberg plugin, which hooks into the `render_block` filter via `gutenberg_restore_list_item_inner_blocks_after_binding()`. The callback:
- Verifies the block is `core/list-item` and contains `$instance->inner_blocks`.
- Checks for a `content` binding in the block's `metadata` (including `__default` bindings from `core/pattern-overrides`) using the helper `gutenberg_list_item_metadata_has_content_binding()`.
- Renders all inner blocks and checks if their HTML already exists in the filtered `$block_content`.
- If missing, it locates the closing `</li>` tag and injects the inner block HTML before it.
- PHPUnit tests in `phpunit/block-bindings-test.php` validate preservation of nested unordered/ordered lists, deeply nested structures, non-list inner blocks (e.g., `core/button`), and pattern override bindings. E2E tests in `test/e2e/specs/editor/various/block-bindings/custom-sources.spec.js` verify editor behavior and frontend rendering.

## Contribution

Opened and merged by @cbravobernal as a compatibility shim for the broader List Item block bindings rollout. Review focused on verifying the inner-block re-append logic against Core's 7.1 implementation, and the author noted the workaround is already approved on the backport and will be removed once the Gutenberg plugin raises its minimum WordPress version to 7.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
