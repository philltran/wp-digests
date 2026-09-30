# #81977: Media: Attach unattached media to the current post on save/publish

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Feature] Media`, `[Package] Editor`
- **Merged:** [`e305ee3`](https://github.com/WordPress/gutenberg/commit/e305ee351ec2bd812cc5c0d58614c3a2dbd8154a)
- **Discussion:** [#81977](https://github.com/WordPress/gutenberg/pull/81977) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg `editor` package's `savePost` action now attaches unattached media to the post after a successful save. It collects the media IDs used by Image and Gallery blocks in the post's own blocks and sets `post` (the attachment parent) on any attachment that has no parent. This narrows a long-standing gap with the classic editor, where picking an existing library image auto-attached it. It is on by default and can be turned off with a new `autoAttachMediaEnabled` editor setting.

## Impact

- **Site owners / editors:** After saving or publishing, images already in the media library that appear in the post and have no parent become attached to that post. The dynamic Gallery block ("Use attached images") and the inserter's attached-images category then refresh without an editor reload. Media already attached to another post is left untouched.
- **Plugin & theme developers:** The behavior is on by default in the block editor. If you don't want it, set `autoAttachMediaEnabled: false` in the editor settings, for example via the `block_editor_settings_all` filter or `updateEditorSettings`. Code that depends on `post_parent` being 0 for library-selected media will now see the parent set after a block-editor save.
- **Headless & REST consumers:** No server-side change. Saving through the REST API or WP-CLI does not trigger attachment, because the logic is client-side only. After an editor save, expect extra `PUT /wp/v2/media/<id>` requests with `post: <postId>`.
- **Limits:** Posts with more than 100 distinct media IDs are skipped entirely (a `console.warn` is logged). Only viewable post types are processed. Autosaves, previews, trashing, and failed saves are excluded.

## Technical details

**`packages/editor/src/store/actions.js` (`savePost`)**

- Snapshots `select.getEditorBlocks()` as `savedBlocks` alongside the content it sends, so media added while the save request is in flight waits for the next save.
- After `REQUEST_POST_UPDATE_FINISH`, calls `attachMediaInPost( registry, { id, type, blocks } )` without awaiting it. The call is guarded by:
  - no `error`
  - not `options.isAutosave`
  - not `options.isPreview`
  - `! select.isDeletingPost()` (this catches `trashPost`, which deletes the post and then calls `savePost`)
  - `previousRecord.status !== 'trash'`
  - `select.getEditorSettings().autoAttachMediaEnabled`

**`packages/editor/src/store/defaults.js`**

- Adds `autoAttachMediaEnabled: true` to `EDITOR_SETTINGS_DEFAULTS`, with a JSDoc typedef entry.

**New `packages/editor/src/store/utils/attach-media-in-post/index.js`**

- Extracts IDs via `getMediaIdsInBlocks`. The implementation is in a file truncated from the diff, so its exact block handling is not shown here.
- Returns early if there are no IDs, or if there are more than `MAX_MEDIA_TO_ATTACH` (100). The cap matches the REST `per_page` maximum and the Gallery dynamic source's `MAX_IMAGES`.
- Resolves `getPostType( postType )` and bails unless `viewable`, which excludes templates and similar types that `savePost` also saves.
- Fetches `getEntityRecords( 'postType', 'attachment', { include: mediaIds, per_page: 100 } )` and filters to items with no `post`. It never overwrites an existing parent.
- Runs `saveEntityRecord( 'postType', 'attachment', { id, post: postId } )` for each via `Promise.allSettled`, so one failure doesn't block the rest. Failures are logged with `console.warn`; nothing is surfaced to the user.
- If at least one save succeeded, calls `invalidateAttachmentResolutions( registry )` to clear cached media lists. The diff was truncated before that file's body.
- The whole routine is wrapped in try/catch, so errors can't fail the save.

The new tests in `actions.jsdom.test.js` cover:

- attaching media the post displays
- ignoring media that belongs to the template (with "Show template" on, the canvas holds the template tree with the post inside `core/post-content`)
- ignoring media added mid-save
- the setting being off
- trashing
- autosave

The CHANGELOG entry is filed under Bug Fixes.

## Contribution

Andrew Serong opened this as a narrower alternative to the earlier attempts #81804 and #81856, which he described as either too greedy or too verbose in the UI. It addresses #66663 and #81491. The design debate centered on client-side JS versus PHP. He chose JS because the editor save signals that the user authored the content, it allows a setting toggle, and it allows cache invalidation so the editor refreshes without a reload. Review feedback led to the 100-image cap and the template-canvas fix, and a reviewer said it had been honed down to the basics. Serong still has misgivings about core's attachment-relationship model but considers this the intended scope. The PR notes Claude Code was used.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
