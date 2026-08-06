# #79934: Visual revisions: add shareable urls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @priethor
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Editor`, `[Feature] History`, `[Package] Edit Post`, `[Package] Edit Site`, `Backported to WP Core`, `[Package] Router`
- **Merged:** [`816fdbb`](https://github.com/WordPress/gutenberg/commit/816fdbb14f353498aed164cde410274d741057ec)
- **Discussion:** [#79934](https://github.com/WordPress/gutenberg/pull/79934) · 13 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The post and site editors now support deep-linking to specific revisions via a `revision` query parameter. When a URL contains `revision=<id>`, the editor opens directly in visual revisions mode at that snapshot, and navigating revisions updates the address bar in real time. This eliminates the need to manually scrub through the revision slider to share or reference a specific state.

## Impact

- **Plugin & theme developers:** No action required. The `revision` query parameter is handled internally by the editor packages.
- **Site owners & editors:** Can now bookmark or share direct links to specific post/site revisions.
- **Hosting & platform teams:** No configuration changes needed. The feature respects existing `disableVisualRevisions` settings and falls back to the classic revision screen when visual revisions are disabled or unavailable.
- **Breaking changes/Deprecations:** None. The internal `getPostEditURL` signature now accepts an optional `revisionId` parameter, but this does not affect public APIs or external integrations.

## Technical details

The diff introduces URL synchronization for revisions across both the post and site editors. In `packages/edit-post/src/components/browser-url/index.js`, the `BrowserURL` component now reads the initial `revision` query arg via `getQueryArg`, calls `openRevision` to load the snapshot, and syncs `currentRevisionId` to the address bar using `window.history.replaceState`. Rapid slider movements are throttled with a 300ms debounce (`URL_WRITE_DEBOUNCE_MS`) to prevent Safari’s 100 History API calls/30s limit. A new `useClassicRevisionRedirect` hook intercepts deep links when `disableVisualRevisions` is true, validates the revision via `registry.resolveSelect( coreStore ).getRevision`, and redirects to `revision.php` if visual revisions are unavailable.

In `packages/edit-site/src/components/editor/use-revisions-url-sync.js`, a similar `useRevisionsURLSync` hook syncs the `revision` param using `@wordpress/router`’s `useHistory` and `useLocation` private APIs, applying the same debounce logic.

A critical resolver fix in `packages/core-data/src/resolvers.js` ensures `getRevisions` properly awaits `dispatch.receiveRevisions` and calls `dispatch.finishResolutions` outside the batch, preventing race conditions where `getRevision` would trigger redundant API fetches after a bulk load.

**Before/after URL generation pattern:**
```diff
-export function getPostEditURL( postId ) {
-	return addQueryArgs( 'post.php', { post: postId, action: 'edit' } );
+export function getPostEditURL( postId, revisionId ) {
+	const args = { post: postId, action: 'edit' };
+	if ( revisionId ) {
+		args.revision = revisionId;
+	}
+	return addQueryArgs( 'post.php', args );
 }
```

## Contribution

Opened by @priethor as part of the visual revisions initiative (#79120), the PR was co-authored by several core contributors. The implementation required balancing real-time URL updates with browser history limits, leading to the 300ms debounce strategy to prevent Safari’s 100 History API calls/30s limit. After review cycles and a minor bot-labeling correction, it was merged and backported to WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
