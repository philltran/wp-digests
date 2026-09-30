# #82465: Query Loop: Fall back to `post` when the query has no `postType`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Query Loop`, `Backported to WP Core`
- **Merged:** [`b44a056`](https://github.com/WordPress/gutenberg/commit/b44a056ee1f5ae7000644ece47e872997766fd5d)
- **Discussion:** [#82465](https://github.com/WordPress/gutenberg/pull/82465) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Query Loop editor UI no longer crashes when a block's `query` attribute has no `postType`. The inspector controls and the Post Template edit component now default `postType` to `'post'`, which matches what `build_query_vars_from_query_block()` already does on the front end. The PR also adds a missing `?.` guard on `labels` in the inspector and documents that a variation's `query` replaces the block's default `query` object.

## Impact

**Plugin & theme developers**
- Query Loop variations that set `query` without `postType` (e.g. `{ perPage: 3, inherit: false }`) previously crashed the block inspector, and the Post Template preview never loaded. They now work and show `Post` as the post type.
- The docs now state that a variation's `query` replaces the default object rather than merging into it. Set `postType` and every other property your variation relies on.
- The same markup that renders posts on the front end now renders posts in the editor, so the two match.

**Site owners / editors**
- Existing content with a `query` lacking `postType` no longer breaks the editor.

**Backport**
- The crash first appeared in WordPress 7.1 (via #64916). The PR is labeled `Backported to WP Core` and was added to the backport tracking list (#82383) for 7.1.1.

No code changes are required, though variation authors should still set `postType` explicitly.

## Technical details

The diff makes three code changes and one docs change:

- `packages/block-library/src/query/edit/inspector-controls/index.jsx`: `postType` in the destructured `query` now defaults to `'post'`.
- Same file: `getPostType( postType )?.labels.singular_name` becomes `getPostType( postType )?.labels?.singular_name`. This is the missing guard the discussion identified as the root cause of the crash.
- `packages/block-library/src/post-template/edit.jsx`: `postType` in the destructured `query` defaults to `'post'`, so the editor's post fetch uses the same default as PHP.
- `docs/how-to-guides/block-tutorial/extending-the-query-loop-block.md` gains a paragraph on `query` replacement semantics. `packages/block-library/CHANGELOG.md` gains an entry.

```js
// before
const { postType, perPage } = query;
// after
const { postType = 'post', perPage } = query;
```

The change is client-side only. It does not modify the saved `query` attribute, PHP, hooks, or REST behavior. The front end is untouched because `build_query_vars_from_query_block()` already queried posts in this case.

## Contribution

The regression was reported in issue #82453. @t-hamano noted in review that the missing `?.` guard was the root cause, that the problem began in WordPress 7.1 with #64916, and that the fix should go into 7.1.1. @adamsilverstein agreed and said it would be cherry-picked before RC via the tracking PR #82383. The PR description notes it was developed with AI tooling under the author's direction and review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
