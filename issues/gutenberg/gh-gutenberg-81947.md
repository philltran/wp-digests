# #81947: Editor: Store the finalize response as the attachment record

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Editor`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`a474f40`](https://github.com/WordPress/gutenberg/commit/a474f40496568a882f6f93e5ff4461bf43a8cbbe)
- **Discussion:** [#81947](https://github.com/WordPress/gutenberg/pull/81947) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

After a client-side media upload, the editor now stores the attachment record returned by the `finalize` endpoint in core-data instead of invalidating and refetching the attachment. `finalize` is the point where `media_details.sizes` gets populated, so a refetch issued earlier, or served from a URL-keyed host cache, could leave the editor holding a record with `"sizes": {}`. The Image block's Resolution control then never appeared. Skipping the refetch removes that failure mode and saves a request per upload. The PR also makes the Gutenberg `finalize_item()` re-read the post before building its response.

## Impact

- **Site owners / editors:** Fixes the Image block's Resolution control missing after a client-side upload on hosts that cache REST responses by URL, or under out-of-order responses (issue #81844). Confirmed by the reporter to fix the issue. The PR discussion targets the 7.1.1 minor release.
- **Plugin & theme developers:** No API changes. Code that relied on an attachment `GET` after `finalize` will no longer see one. Uploads processed by the server (video, audio, PDFs, images too large for wasm-vips) never call `finalize` and refresh as before.
- **Hosting & platform:** A REST cache keyed on the full request URL, even for authenticated `no-cache` responses, no longer breaks client-side uploads, because the post-finalize refetch is gone. Other `invalidateResolution` callers can still hit the same caching problem.
- **Plugins hooking `wp_generate_attachment_metadata`:** The finalize response now reflects post-row changes made in that filter (e.g. a changed `post_mime_type`). If a callback deletes the attachment, `finalize` now returns a `WP_Error` instead of a 200.
- **Action required:** None for most sites. The server-side re-read only applies when the Gutenberg plugin is active, since `lib/media/load.php` swaps in the Gutenberg controller. A core equivalent is proposed in wordpress-develop#13412 / Trac #66056.

## Technical details

**New module** `packages/editor/src/utils/media-upload/finalized-attachments.ts`:
- `receiveFinalizedAttachment( record )` returns early if `record?.id` is falsy. Otherwise it calls `dispatch( coreDataStore ).receiveEntityRecords( 'postType', 'attachment', record, query )` for `{ context: 'view' }` and for `undefined`, matching the two resolution keys the editor uses. The record is passed as a single object, not an array, so core-data treats it as a single-item receive and list queries (media library, inserter Media tab) are untouched. It then adds `record.id` to a module-level `Set`.
- `consumeFinalizedAttachment( id )` returns `finalized.delete( id )`.

**`media-finalize/index.js`:** `mediaFinalize` returns `undefined` on an empty response. Otherwise it calls `receiveFinalizedAttachment( response )` with the untransformed REST record, then returns `transformAttachment( response )` as before.

**`media-upload/on-success.js`:** `mediaUploadOnSuccess` now skips attachments with no `id` or where `consumeFinalizedAttachment( id )` is true. Everything else still gets `invalidateResolution( 'getEntityRecord', [...] )` with and without `{ context: 'view' }`.

```js
// before
for ( const a of attachments ) {
	if ( a.id ) { invalidateResolution( ...view ); invalidateResolution( ...noQuery ); }
}
// after
for ( const a of attachments ) {
	if ( ! a.id || consumeFinalizedAttachment( a.id ) ) continue;
	invalidateResolution( ...view ); invalidateResolution( ...noQuery );
}
```

The finalize response is prepared in the `edit` context, which the PR describes as a superset of `view`.

**Server, `lib/media/class-gutenberg-rest-attachments-controller.php`:** `finalize_item()` now calls `$this->get_post( $attachment_id )` before `prepare_item_for_response()`. This picks up post-row changes made by `wp_generate_attachment_metadata` callbacks. A `WP_Error` from the re-read is returned to the client.

**Tests and changelog:** Vitest unit tests were added for `finalized-attachments`, `on-success` and `media-finalize`. PHP tests include `test_finalize_response_contains_generated_sub_sizes` and a case for the deleted-attachment path. An e2e spec simulates a URL-keyed cache. There is a `packages/editor/CHANGELOG.md` entry and `backport-changelog/7.1/13412.md`.

## Contribution

Written by @adamsilverstein with Claude Code assistance (disclosed in the PR), as one of several PRs addressing #81844; #81938, #81846 and #81886 were also in play. The PR's own e2e cross-check showed it does not replace #81846 (resolver out-of-order guard) or #81886 (list staleness), since `receiveEntityRecords` bypasses the resolver. The maintainers only wanted this one backported to 7.1: @adamsilverstein was unsure the other two are needed or fix the reporter's problem. @t-hamano asked about shipping it in the 7.1 minor. @gregbenz confirmed the fix. Review feedback led to the server-side re-read in `finalize_item()`, which was then mirrored into a core PR and Trac ticket for the 7.1.1 milestone.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
