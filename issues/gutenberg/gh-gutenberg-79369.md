# #79369: Image block: stop interpreting `$`/`\` in img attributes as regex backreferences

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @lonesahilnazir
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Image`
- **Merged:** [`d64aef5`](https://github.com/WordPress/gutenberg/commit/d64aef59a080ebf925d22f60e70f404d5578ee4e)
- **Discussion:** [#79369](https://github.com/WordPress/gutenberg/pull/79369) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Image block's lightbox rendering no longer mangles `$` and `\` sequences in `<img>` attributes. `block_core_image_render_lightbox()` injected the trigger button with `preg_replace()`, so a value like `alt="Only $1 today"` or a filename containing `$1` was treated as a backreference and silently replaced with an empty string. The function now uses a literal `str_replace()`, so attributes render exactly as authored.

## Impact

- **Site owners / content authors:** Images with lightbox ("Expand on click") enabled and `$N`, `${N}` or `\N` sequences in alt text, title, class or `src` (e.g. prices such as `$1`) previously had those fragments dropped on the front end. They now render intact. No content changes or migration are needed.
- **Plugin & theme developers:** No API, hook or signature changes. Markup for normal images is unchanged. Any code or snapshot tests that (unintentionally) depended on the stripped output for such values would see different output.
- **Hosting / headless / REST consumers:** No action required. The PR notes the output stays attribute-escaped, so this is a correctness fix and not an XSS issue.

## Technical details

**Root cause:** in `block_core_image_render_lightbox()`, the button markup begins with the matched tag (`$img[0]`) and therefore embeds author-controlled attribute values. It was then substituted into the block content with `preg_replace( '/<img[^>]+>/', $button, $body_content )`. In a `preg_replace()` replacement string, `$1`, `${1}` and `\1` are backreferences. A reference to a non-existent capture group expands to an empty string, which broke `alt`/`src`.

**Fix (as described in the PR):** a `core/image` block contains exactly one `<img>`, so a literal replacement suffices:

```php
// Before
$body_content = preg_replace( '/<img[^>]+>/', $button, $body_content );

// After
if ( isset( $img[0] ) ) {
	$body_content = str_replace( $img[0], $button, $body_content );
}
```

The `isset( $img[0] )` guard skips the replacement when no `<img>` match exists.

**Test:** adds a PHPUnit regression test, `test_should_not_treat_dollar_sequences_in_img_as_backreferences_in_lightbox`. It renders a lightbox-enabled image whose alt text and src filename contain `$1`, and asserts the sequences survive. It renders through `WP_Block::render()` because the button is added via the `render_block_core/image` filter, which only runs on the full render path. Note that the diff itself was not available in the input (the review bot skipped the 2,970-file PR), so the code above reflects the PR description rather than a reviewed diff.

## Contribution

The PR was opened by @lonesahilnazir to close #79368, with Claude used to help draft the description and regression test. It sat without review for a while, prompting a follow-up ping from the author, and a bot flagged a missing `[Type]` label until labels were added. CodeRabbit skipped review because the PR was unusually large (2,970 files), which points to a branch/base problem rather than the actual change. The props bot listed @youknowriad alongside the author.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
