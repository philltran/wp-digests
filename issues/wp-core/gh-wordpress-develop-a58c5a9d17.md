# HTML API: Prevent `set_modifiable_text()` from abruptly closing comments.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-09-17
- **Commit:** [`a58c5a9d17`](https://github.com/WordPress/wordpress-develop/commit/a58c5a9d1782b35f869252bce93523159b179a52)
- **Usefulness:** 3/5

## Summary

`WP_HTML_Tag_Processor::set_modifiable_text()` now rejects replacement text that would abruptly close an HTML comment when it is placed at the start of the comment. Previously the guard only caught `-->` and `--!>` anywhere in the string. It missed `>` and `->` at the very start, which the HTML parser treats as an abruptly closed empty comment when they directly follow `<!--`.

## Impact

- **Plugin & theme developers:** Code that calls `set_modifiable_text()` on an HTML comment token with text beginning with `>` or `->` now triggers `_doing_it_wrong()` and the call returns `false` instead of writing text that would truncate the comment. Text that is otherwise safe is unaffected.
- **Security-minded integrators:** This closes a gap where untrusted text written into a comment could end the comment early, so the remainder is parsed as markup.
- **Site owners / hosting / REST consumers:** No action required.

## Technical details

The change is a single regex edit in `src/wp-includes/html-api/class-wp-html-tag-processor.php`, inside the comment branch of `set_modifiable_text()` (the check gated on `self::COMMENT_AS_HTML_COMMENT === $this->comment_type`):

```php
// Before
if ( 1 === preg_match( '/--!?>/', $plaintext_content ) ) {
// After
if ( 1 === preg_match( '/^-?>|--!?>/', $plaintext_content ) ) {
```

The new alternative `^-?>` matches text starting with `>` or `->`. Per the HTML spec, `<!-->` and `<!--->` are abruptly closed comments, so those strings at the start of comment content would end the comment immediately. The existing `--!?>` alternative still handles `-->` and `--!>` anywhere in the text. The error path is unchanged: it raises `_doing_it_wrong()` with 'Comment text cannot contain a comment closer.' No new hooks or API surface. The diff shown includes no test changes.

## Contribution

The record carries only props (jeremyfelt, jonsurrell) and the SVN sync to trunk r63656, with no discussion detail.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
