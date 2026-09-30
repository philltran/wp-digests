# #82517: Core Data: Scope revisionId to the entity it is provided for

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Package] Editor`, `[Feature] History`
- **Merged:** [`817dc1c`](https://github.com/WordPress/gutenberg/commit/817dc1c2867f17b87bb96afb9d5f798e21df276c)
- **Discussion:** [#82517](https://github.com/WordPress/gutenberg/pull/82517) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`EntityProvider` now stores `revisionId` per entity kind and name (`context.revision[kind][name]`) instead of as a single flat context value, and `useEntityProp` applies it only to the record the provider provides. Previously, entering revisions mode on a post made every other entity rendered inside it read from that post's revision, so a Query Loop in the revised post showed `(no title)` for each listed post and fired a bogus request per post to `/wp/v2/posts/<loop post>/revisions/<revised post revision>`. The revision read also switches from `getRevisions` to `getRevision`, which avoids an extra fetch of every revision.

## Impact

**Site owners / editors**
- Query Loops (and any other nested entity) inside a post being viewed in the revisions screen now show their real data instead of `(no title)`, and the spurious per-post revision requests stop.

**Plugin & theme developers**
- Blocks that call `useEntityProp` for `root/site`, `root/comment`, `taxonomy/*`, or another record of the same kind via the `_id` argument no longer inherit the revised post's revision ID.
- Blocks reading the revised post itself (title, excerpt, content, footnotes meta) behave as before.
- Code that passes `revisionId` to `EntityProvider` needs no change; only its scope narrows. Anything that read `context.revisionId` directly from `EntityContext` would no longer find it, but the PR states only `useEntityProp` consumed it.

No action required for most developers.

## Technical details

**`packages/core-data/src/entity-provider.jsx`**
- The flat `revisionId` spread is replaced by a nested map mirroring the existing id map. It is only written when `revisionId !== undefined` and `kind` is set:

```js
revision: {
	...parent?.revision,
	[ kind ]: {
		...parent?.revision?.[ kind ],
		[ name ]: revisionId,
	},
}
```

**`packages/core-data/src/hooks/use-entity-prop.js`**
- `revisionId` is now resolved as `context?.revision?.[ kind ]?.[ name ]` only when `String( id ) === String( providerId )` (loose comparison, since IDs can be strings or numbers). This is what excludes Query Loop posts, which share kind and name with the revised post but not its ID.
- The read path uses `select( STORE_NAME ).getRevision( kind, name, id, revisionId, REVISION_QUERY )` rather than `getRevisions` with `per_page: -1` plus a `find` on the revision key. The old collection query did not match the editor's paginated revisions query, so it triggered its own fetch of all revisions. The code comment about a `getRevision` race is removed as stale.
- `REVISION_QUERY` is a module-level constant (`context: 'edit'`, `_fields: 'id,date,author,meta,title,excerpt,content.raw'`) because `getRevision` memoizes on query identity. `content.raw` is requested rather than the whole `content` field to avoid pulling `content.rendered`.
- Value extraction: if the prop is a non-null object with a `raw` key, `value` is `propValue.raw`; otherwise the value is the prop itself. `fullValue` is the prop as returned. An undefined prop returns `{}`.
- The `DEFAULT_ENTITY_KEY`/`getEntityConfig` revision-key lookup is no longer needed and is removed.

**`packages/editor/src/store/private-selectors.js` / `private-actions.js`**
- `buildRevisionsPageQuery` and `restoreRevision` request `title` and `excerpt` instead of `title.raw` and `excerpt.raw`, so the editor's paginated query shares the same fields as `REVISION_QUERY` and the store is hit rather than refetched.

**Tests / changelog**
- Adds an e2e spec, 'Post revisions with nested entities', in `test/e2e/specs/editor/various/revisions.spec.js`, asserting the Query Loop shows the revised post's title changing while the other post stays fixed. A `CHANGELOG.md` entry is added to `packages/core-data`.

## Contribution

The problem was noticed by @tyxla during review of #82275. @ntsekouras flagged a related symptom while testing this PR, and @Mamaduka initially couldn't reproduce it until finding that the Query Loop must include the post whose revision is being viewed. Two follow-up commits fixed a `getRevision` query-identity issue locally (by passing a stable query object) and the current post's Post Title rendering as `(no title)` inside the loop. Mamaduka noted that the general problem of `getRevision`/`getEntityRecord` with an inline-object `query` remains and would be addressed separately. The PR notes it was assisted by Claude.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
