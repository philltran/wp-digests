# #80318: Table of Contents: render front-end list from current post headings

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @getdave
- **Labels:** `[Package] Block library`, `[Type] Feature`, `[Block] Table of contents (experimental)`
- **Merged:** [`85b9c5f`](https://github.com/WordPress/gutenberg/commit/85b9c5f3e92da3179f4682a39510a6b69c64a98a)
- **Discussion:** [#80318](https://github.com/WordPress/gutenberg/pull/80318) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Table of Contents block now dynamically renders its front-end list from the current post's headings instead of relying on the `headings` attribute saved in block markup. This prevents stale navigation when post content is edited outside the block editor or modified by other plugins. The change applies specifically during the `the_content` filter, scanning only direct `core/heading` blocks in the post content.

## Impact

- **Plugin & theme developers**: The experimental Table of Contents block now reflects live post content on the front end. If you were relying on the saved `headings` attribute for front-end rendering, it is now ignored during `the_content`.
- **Site owners**: ToC lists will automatically update when headings are added, removed, or reordered in the post content, without requiring a block re-save.
- **No action required**: Existing posts and blocks remain compatible; the editor still serializes `headings` for back-end consistency.

## Technical details

The primary change occurs in `packages/block-library/src/table-of-contents/index.php`. The `block_core_table_of_contents_render` function now checks `in_array( 'the_content', $wp_current_filter, true )`. When true, it bypasses the saved markup and invokes a new dynamic rendering pipeline:
- `block_core_table_of_contents_get_headings_from_content` uses `WP_Block_Processor` to iterate through post content, normalizing `<!--nextpage-->` markers as `core/nextpage` blocks to track pagination state.
- `block_core_table_of_contents_get_heading_from_block` renders each `core/heading` block, extracts its `id` and text via `WP_HTML_Tag_Processor`, and applies `html_entity_decode` followed by `esc_html` to prevent double-escaping.
- `block_core_table_of_contents_get_heading_link` constructs page-aware fragment links (e.g., `?page=2#heading-id`) using `get_permalink` and `get_query_var( 'page' )`.
- `block_core_table_of_contents_linear_to_nested_heading_list` and `block_core_table_of_contents_build_list_items` recursively structure the flat heading array into nested `<ol>`/`<ul>` markup.
- A new constant `BLOCK_CORE_TABLE_OF_CONTENTS_DEFAULT_HEADING_LEVEL` (value `2`) replaces hardcoded defaults.
- Headings without an `id` render as unlinked `<span class="wp-block-table-of-contents__entry">` elements.

**Before/after behavior**:
```php
// Before: Rendered saved $attributes['headings'] markup directly.
// After: Scans $post->post_content during the_content, ignoring saved headings.
```

## Contribution

Opened by @getdave as part of issue #42229, this PR was merged following minimal review. The author explicitly scoped the implementation to direct `core/heading` blocks in post content, deferring support for synced patterns, template parts, and non-heading block sources to follow-ups. Pagination handling and entity escaping were refined during review to ensure links resolve correctly across paginated posts. Co-authored with @scruffian.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
