# #64916: Add option to exclude current post from query block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @g-elwell
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `First-time Contributor`, `[Block] Query Loop`
- **Merged:** [`8be2862`](https://github.com/WordPress/gutenberg/commit/8be2862a960476cbb0ad6d27c7089929099a47c7)
- **Discussion:** [#64916](https://github.com/WordPress/gutenberg/pull/64916) · 36 comments · 6 reactions
- **Usefulness:** 4/5

## Summary

This PR introduces a toggle to the Query Loop block that excludes the current post from its results. It addresses a frequent design requirement where authors want to display an "Other posts" or "Related posts" section below a single post without duplicating the current post’s title or metadata. The feature works dynamically in both the block editor preview and on the frontend.

## Impact

- **Block theme authors & site builders:** Can now natively prevent the current post from appearing in a Query Loop on singular templates (e.g., `single.html`) without custom CSS or complex query overrides.
- **Plugin & theme developers:** No breaking changes. The frontend behavior relies on a new filter in a compatibility layer, so it requires WordPress 7.1+ or the Gutenberg plugin to function on the frontend.
- **No action required** for existing sites or blocks that do not use the new toggle.

## Technical details

- Adds the `excludeCurrent` attribute (default `null`) to the Query Loop block (`packages/block-library/src/query/block.json`).
- Extends block context: `postType` is added to the Query block’s `usesContext`, and `postId` is added to the Post Template block’s `providesContext` (`packages/block-library/src/post-template/block.json`).
- **Editor preview:** `packages/block-library/src/query/edit/query-content.js` calculates `shouldExcludeCurrentPost` based on `isSingular`, `!inherit`, and post-type matching. If enabled, `packages/block-library/src/post-template/edit.js` dynamically appends the current `postId` to `query.exclude` to filter the editor preview.
- **Inspector UI:** `packages/block-library/src/query/edit/inspector-controls/index.js` renders a `ToggleControl` labeled "Exclude current" inside the Query settings panel, conditionally shown only when `shouldExcludeCurrentPost` is true.
- **Frontend PHP:** A new compatibility file (`lib/compat/wordpress-7.1/query-block.php`) hooks `gutenberg_filter_query_block_exclude_current` to the `query_loop_block_query_vars` filter. When `$block->context['query']['excludeCurrent']` is truthy, it appends `get_the_ID()` to `$query['post__not_in']`.
- Example frontend query modification:
  ```php
  // Before: $query = [ 'post_type' => 'post', 'posts_per_page' => 3 ];
  // After (when toggle is on): $query = [ 'post_type' => 'post', 'posts_per_page' => 3, 'post__not_in' => [ 123 ] ];
  ```

## Contribution

Opened by @g-elwell and merged following extensive review. The PR was co-authored by @t-hamano, @ramonjd, @carolinan, @ntsekouras, @spacedmonkey, @jasmussen, @jameskoster, @ryanwelcher, and others. Reviewers debated whether to use a toggle group (like the Sticky posts control) or a standalone toggle, ultimately settling on a standalone `ToggleControl` for simplicity. The team also agreed to conditionally hide the control outside singular templates and implement the frontend logic via a filter on `query_loop_block_query_vars` in a WordPress 7.1 compat file to avoid core WP version conflicts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
