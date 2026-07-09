# #74866: Latest Posts: Parse blocks in full content display

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @getdave
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Latest Posts`
- **Merged:** [`3e2f88a`](https://github.com/WordPress/gutenberg/commit/3e2f88a476a40f30f3d0a95e32d9c5269a91e1bc)
- **Discussion:** [#74866](https://github.com/WordPress/gutenberg/pull/74866) · 16 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

When "Show full post" is enabled in the Latest Posts block, embedded blocks within displayed posts were previously output as raw markup without being parsed. This change introduces `do_blocks()` processing for full post content, ensuring videos, galleries, and other nested blocks render with proper constraints and styles. It also adds recursion protection to prevent memory exhaustion when a Latest Posts block inadvertently lists itself.

## Impact

- **Block/Theme Developers:** The `displayPostContentRadio='full_post'` mode now outputs fully parsed block markup instead of raw HTML. Developers relying on predictable raw HTML output in this specific context may need to adjust expectations or switch to excerpts.
- **Site Owners:** Existing "Show full post" configurations remain compatible, but layout rendering (especially for videos and galleries) will be corrected to respect container widths.
- **No action required** for upgrading or deployment; this is a transparent bug fix that improves frontend rendering fidelity.

## Technical details

The core behavior change lives in `packages/block-library/src/latest-posts/index.php`. The original `foreach ( $recent_posts as $post )` loop was replaced with a standard `while ( $query->have_posts() ) { ... the_post(); }` iteration to correctly establish the global `$post` context. For full post content, the diff replaces raw HTML extraction (`$post->post_content`) with a call to a new internal function: `_gutenberg_apply_content_filters( $post_content, 'latest-posts' )`. This helper runs `shortcode_unautop()`, `do_shortcode()`, `do_blocks()`, and other standard `the_content` filters.

To prevent infinite recursion (and the resulting memory exhaustion reported during testing), a static `$rendering_stack` array tracks post IDs currently being rendered. If `get_the_ID()` exists in the stack, subsequent rendering is skipped. The global `$post` object is manually saved before the loop and restored afterward using `setup_postdata()`, replacing the previous reliance on implicit context handling.

An edit preview adjustment was made in `packages/block-library/src/latest-posts/edit.js`, switching `post.content.raw.trim()` to `post.content.rendered.trim()` to match the parsed output. The `template-part/index.php` render callback was also updated to use the new `_gutenberg_apply_content_filters()` helper.

## Contribution

Opened by @getdave, this PR evolved from earlier investigation by @sabbir1991 regarding root causes in the content rendering pipeline. Following testing by @MaggieCabrera that uncovered a memory exhaustion issue when the block displayed itself, recursion safeguards were added to prevent infinite loops. The implementation was finalized with input from @scruffian and multiple core contributors during review, merging as `3e2f88a`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
