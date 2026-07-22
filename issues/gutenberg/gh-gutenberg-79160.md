# #79160: Editor: Guard PostViewLink against post types without a labels object

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mayank-Tripathi32
- **Labels:** `[Type] Bug`, `[Package] Editor`, `[Package] Block library`
- **Merged:** [`6cf51dc`](https://github.com/WordPress/gutenberg/commit/6cf51dc13eb92e57f6f88c5d5f82194d5fa1325b)
- **Discussion:** [#79160](https://github.com/WordPress/gutenberg/pull/79160) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor now safely handles post types registered without a `labels` object, preventing a `TypeError` crash in the `PostViewLink` component and related UI elements. Previously, accessing `postType?.labels.view_item` would throw when `labels` was undefined, which occurs for post types like `wp_template_part` or custom types missing the `labels` argument. The fix extends optional chaining to `postType?.labels?.view_item` across five editor and block-library files, falling back to default labels when the object is absent.

## Impact

- **Plugin & theme developers:** No action required. If you register a custom post type without a `labels` argument, the editor will no longer crash when viewing or editing that post.
- **Site owners:** Unaffected unless using a theme/plugin that registers post types without `labels`; the editor will now render gracefully instead of throwing a console error and breaking the UI.
- **Headless & REST consumers:** No impact; this is strictly a client-side editor rendering fix.

## Technical details

The diff modifies five JavaScript files to extend optional chaining on `labels` properties accessed via the `getPostType` store selector:
- `packages/editor/src/components/post-view-link/index.js`: `postType?.labels.view_item` → `postType?.labels?.view_item`
- `packages/editor/src/components/post-url/index.js`: `postType?.labels.view_item` → `postType?.labels?.view_item`
- `packages/block-library/src/post-date/edit.js`: `postType?.labels.singular_name` → `postType?.labels?.singular_name` (two instances)
- `packages/block-library/src/post-featured-image/edit.js`: `postType?.labels.singular_name` → `postType?.labels?.singular_name`
- `packages/editor/src/components/post-card-panel/index.js`: `labels?.name.toLowerCase()` → `labels?.name?.toLowerCase()`

Before:
```js
label: postType?.labels.view_item,
```
After:
```js
label: postType?.labels?.view_item,
```
The change also introduces a new unit test suite for `PostViewLink` and adds a regression test to `PostTypeSupportCheck` to verify it returns `null` when `supports` is missing. No hooks, filters, or REST schema changes are involved.

## Contribution

Opened by @Mayank-Tripathi32 to resolve a crash reported in #62918. While implementing the fix, the author discovered the same unsafe optional-chaining pattern in four other editor components and proposed extending the guard. @Mamaduka reviewed the additional locations, noting that while `context=edit` REST calls typically supply default labels, the defensive checks are harmless and prevent edge-case crashes. The PR was merged with co-authors @Mamaduka and @JNBG after unit tests were added.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
