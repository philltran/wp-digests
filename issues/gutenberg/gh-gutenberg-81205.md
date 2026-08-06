# #81205: Visual revisions: add shareable urls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Editor`, `[Feature] History`, `[Package] Edit Post`, `[Package] Edit Site`, `[Package] Router`
- **Merged:** [`6d6d5b8`](https://github.com/WordPress/gutenberg/commit/6d6d5b86ac9015be7f1f12aecff6d1bf89821452)
- **Discussion:** [#81205](https://github.com/WordPress/gutenberg/pull/81205) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This change adds shareable URLs for visual revisions by appending a `revision` query parameter to post and site editor URLs. It also fixes a race condition in the `core-data` revision resolvers where individual `getRevision` resolutions could complete before batched revisions were stored, causing redundant API fetches. The feature enables direct linking to specific revision previews and improves editor stability in Safari by debouncing rapid History API calls.

## Impact

- **Plugin & theme developers**: No breaking changes. If you build custom editor UIs or parse the post edit URL, note that `revision` is now appended when previewing a revision.
- **Site owners & editors**: Can now bookmark or share direct links to specific visual revision previews.
- **Hosting & platform**: No action required. The debounced URL writes prevent Safari crashes from rapid History API calls.
- **Core data consumers**: The `getRevisions` resolver behavior is corrected; no migration needed.

## Technical details

- `packages/edit-post/src/components/browser-url/index.js`: `getPostEditURL(postId)` now accepts an optional `revisionId` and appends `&revision=<id>` to `post.php`. The `BrowserURL` component reads the initial `revision` query arg via `getQueryArg`, calls `openRevision(initialRevisionId)`, and debounces `window.history.replaceState` calls using a 300ms `URL_WRITE_DEBOUNCE_MS` threshold to avoid Safari's 100-calls/30s limit.
- `packages/edit-site/src/components/editor/use-revisions-url-sync.js`: New hook that syncs the `revision` query parameter to the site editor's router history, mirroring the post editor behavior.
- `packages/core-data/src/resolvers.js`: The `getRevisions` resolver was refactored to `await dispatch.receiveRevisions(...)` and moved `dispatch.finishResolutions('getRevision', resolutionsArgs)` outside `registry.batch`. This ensures individual `getRevision` resolutions are marked complete only after the batched revisions are stored, preventing redundant fetches.
- `packages/edit-post/src/components/browser-url/use-classic-revision-redirect.js`: New hook that checks `editor.getEditorSettings().disableVisualRevisions` and redirects to `revision.php` if visual revisions are disabled, ensuring legacy revision screens load correctly.

**Before/after usage pattern:**
```js
// Before
export function getPostEditURL( postId ) {
	return addQueryArgs( 'post.php', { post: postId, action: 'edit' } );
}

// After
export function getPostEditURL( postId, revisionId ) {
	const args = { post: postId, action: 'edit' };
	if ( revisionId ) {
		args.revision = revisionId;
	}
	return addQueryArgs( 'post.php', args );
}
```

## Contribution

Backported from #79934 to the `wp/7.1` branch, the change was cherry-picked with only `CHANGELOG` file conflicts. The author validated it via smoke testing and merged after CI passed. The PR thread contains no recorded design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
