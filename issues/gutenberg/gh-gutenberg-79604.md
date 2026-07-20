# #79604: Notes: Add @mention autocomplete

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Status] In Progress`, `[Package] Editor`, `[Type] Feature`, `[Package] DataViews`, `[Feature] Notes`
- **Merged:** [`8f3a7e7`](https://github.com/WordPress/gutenberg/commit/8f3a7e7176b6745e29a17a6e5f82c8c8cf907f4e)
- **Discussion:** [#79604](https://github.com/WordPress/gutenberg/pull/79604) · 11 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds `@` mention autocomplete to the Notes feature in the block editor. Typing `@` triggers a keyboard-navigable user list; selecting a user inserts a styled mention chip carrying the user's ID. This enables collaborators to tag specific users within note threads, mirroring common collaboration tools.

## Impact

- **Editor users:** No action required; the feature activates automatically when Notes are enabled.
- **Plugin & theme developers:** The `RichTextControl` assembly in `@wordpress/dataviews` now accepts a `completers` prop, enabling custom autocomplete integrations in rich text fields without duplicating the `@wordpress/components` autocomplete hook.
- **Hosting & platform:** Note content now carries `class` attributes on mention elements. The kses allowlist is strictly scoped to `note` comment types via `preprocess_comment` and `rest_preprocess_comment`, leaving regular comment sanitization unaffected.

## Technical details

- **Rich text autocomplete:** `packages/dataviews/src/components/dataform-controls/richtext/control.tsx` adds an optional `completers` prop to `RichTextControl`. It wires the field to `__unstableUseAutocompleteProps` from `@wordpress/components`, returning zero-cost when omitted.
- **Mention completer:** `packages/editor/src/components/collab-sidebar/note-mention-completer.tsx` implements the `@` trigger. Per review, it inserts a `<span class="wp-note-mention user-{id}">` chip instead of a link, preserving the user ID for downstream notification logic while avoiding arbitrary navigation.
- **Blur & focus management:** `packages/editor/src/components/collab-sidebar/add-note.js` replaces DOM-class sniffing (`.components-popover`) with a deferred dismissal pattern. It uses `requestAnimationFrame` to delay unmounting and `cancelAnimationFrame` triggered by the container's `onFocus` handler, preventing premature dismissal when focus shifts to portaled popovers or format UIs.
- **Server-side sanitization:** `lib/compat/wordpress-7.1/block-comments.php` introduces `gutenberg_notes_allow_mention_attributes()` to temporarily allow `class` on `<a>` tags in the `pre_comment_content` kses context. It is armed via `gutenberg_notes_scope_mention_kses()` on `preprocess_comment` and `gutenberg_notes_scope_mention_kses_rest()` on `rest_preprocess_comment`, and self-removes at `PHP_INT_MAX` on `pre_comment_content` to avoid skipping subsequent callbacks like `wp_rel_ugc()`.

## Contribution

Opened by @adamsilverstein and merged following review cycles with @fcoveram, @ciampo, and @Mamaduka. The PR landed for beta 1 with several post-merge follow-ups (e.g., Escape key handling, ARIA attributes, overlay placement) acknowledged for beta 2. Co-authored by @talldan, @ciampo, @Mamaduka, @fcoveram, @t-hamano, @jasmussen, and @jeffpaul.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
