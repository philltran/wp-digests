# #79937: Site Editor v2 experiment: hide admin bar in distraction-free mode

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`
- **Merged:** [`0d0c4b8`](https://github.com/WordPress/gutenberg/commit/0d0c4b89033a93ba1a363d98d05930ae28e8d618)
- **Discussion:** [#79937](https://github.com/WordPress/gutenberg/pull/79937) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This change updates the inline CSS for the Site Editor v2 (Extensible Site Editor experiment) to hide the WordPress admin bar when distraction-free mode is active. Previously, the admin bar remained visible and retained its viewport height during this state. The update aligns the site editor's behavior with existing post-editor patterns by conditionally zeroing the `--wp-admin--admin-bar--height` CSS variable and hiding the `#wpadminbar` element via a `body:has()` selector.

## Impact

- **Theme & plugin developers working with the Extensible Site Editor experiment**: No code changes required; visibility is managed entirely through inline CSS injected into the `admin-bar` stylesheet.
- **Site editors using distraction-free mode**: Experience a clean, full-height canvas matching post-editor behavior when switching to distraction-free layouts.
- **No action required** for standard installations unless explicitly enabling and testing the v2 experiment.

## Technical details

The diff modifies `lib/experimental/pages/site-editor.php`. The patch appends a `@media (min-width: 782px)` CSS block to the existing `$css` variable, which is injected via `wp_add_inline_style()` for the `admin-bar` handle. When `.editor-editor-interface.is-distraction-free` is present on an editor component, the selector targets the `<body>` to set `--wp-admin--admin-bar--height: 0px;` and applies `display: none;` to `#wpadminbar`. This logic directly mirrors the layout calculations previously established in the post and site editor packages.

## Contribution

Opened by @fushar, this PR was merged following a straightforward review cycle. The implementation adapts an existing CSS pattern from the core post editor to the newly introduced Site Editor v2 experiment package. A co-author attribution for @tyxla was added during the merge commit.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
