# Removing title attributes in author link functions

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Stephen Bernhardt
- **Published:** 2026-05-14
- **Tags:** `General`, `7.0`, `dev-notes`, `dev-notes-7-0`
- **Link:** [https://make.wordpress.org/core/2026/05/14/removing-title-attributes-in-author-link-functions/](https://make.wordpress.org/core/2026/05/14/removing-title-attributes-in-author-link-functions/)

## Summary

WordPress 7.0 strips default `title` attributes from author-related links to improve markup cleanliness and accessibility, while exposing the removed text via filter parameters for backward-compatible customization. The change affects `get_the_author_link()`, `the_author_link()`, author archive link generation, and `wp_list_authors()`. Title values are no longer injected into the DOM by default, though themes and plugins can opt out or repurpose them through updated hooks and flags.

## Impact

- **Plugin & Theme Developers**:
  - `get_the_author_link()` and `the_author_link()` now accept a `$use_title_attr` parameter (default `true`). Passing `false` suppresses the "Visit Author&#8217;s website" tooltip entirely.
  - The `the_author_posts_link` filter signature expanded to three parameters: `$link`, `$author` (display name), and `$title` (previously used text). Hooks relying on the prior two-parameter contract will lose access to the title text.
  - `wp_list_authors()` no longer outputs `title="Posts by Author"` on list item links.
- **Site Owners / General**: No action required. Default rendering remains functionally identical in terms of link destination and visible text.
- **Required Changes**: Update any custom implementations parsing the `rel` or `title` attributes for author links, and refactor `the_author_posts_link` callbacks to accept three arguments if title repurposing is needed.

## Technical details

The patch modifies author template tag outputs to conditionally exclude `title` attributes:
- `get_the_author_link()` / `the_author_link()` now accept `$use_title_attr`. When `false`, skips injecting the default tooltip text.
- Author archive link generation drops the `title="Posts by [Author]"` attribute by default. The text is no longer embedded in the markup but is passed to the `the_author_posts_link` filter.
- Filter signature update: `apply_filters( 'the_author_posts_link', $link )` becomes `apply_filters( 'the_author_posts_link', $link, $author_display_name, $title_text )`. This enables developers to reconstruct link text using the previously injected title rather than discarding it.

```php
// WordPress 6.9 default output
echo get_the_author_link(); 
// <a href="https://author.example.com" title="Visit Author&#8217;s website" rel="author external">Author</a>

// WordPress 7.0 with opt-out flag
echo get_the_author_link( false );
// <a href="https://author.example.com" rel="author external">Author</a>
```

## Contribution

This change was merged during the WordPress 7.0 development cycle, authored by Stephen Bernhardt with review contributions from @amykamala and @audrasjb. The implementation focuses on stripping default tooltip attributes while preserving the underlying text for developers via the updated `the_author_posts_link` filter signature. The approach was finalized to avoid hard-breaking existing themes that rely on title attributes, instead providing a clean opt-out path and maintaining backward compatibility through parameter expansion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
