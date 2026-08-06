# #80725: Upload Media: Fail the item when the /finalize request fails

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`87ec57d`](https://github.com/WordPress/gutenberg/commit/87ec57d38660f57925025f5b4930ef2b6826e5e7)
- **Discussion:** [#80725](https://github.com/WordPress/gutenberg/pull/80725) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Client-side media processing now correctly fails an upload when the server’s `/wp/v2/media/<id>/finalize` request returns an error, instead of silently reporting success. Previously, a failed finalize was only logged, causing the editor to display a completion message and autosave a block with missing responsive sub-sizes and inconsistent file references. This change ensures that a server-side commit failure surfaces as a user-facing error and cancels the queue item, preventing incomplete attachments from being saved.

## Impact

- **Plugin & theme developers / block authors:** No breaking changes or deprecations. Client-side media uploads (Web Workers) will now correctly surface finalize failures instead of silently succeeding.
- **Site owners / editors:** Uploads that fail during the server-side metadata commit step will now show an error notice instead of a false "Media upload complete" message.
- **Hosting & platform teams:** No configuration changes required. Ensure the `/wp/v2/media/<id>/finalize` endpoint remains stable, as transient 500s will now halt the upload flow rather than being ignored.
- **No action required** for existing codebases; this is a bug fix in the Gutenberg client-side media queue.

## Technical details

The change modifies `packages/upload-media/src/store/private-actions.ts` in the `finalizeItem` thunk. Previously, the `catch` block only logged the error and allowed execution to fall through to `finishOperation`. The diff replaces that with a call to `dispatch.cancelItem()` wrapped in a new `UploadError`, which halts the queue item and prevents `finishOperation` from firing.

Before:
```js
} catch ( error ) {
    // Log but don't fail the upload if finalization fails.
    console.warn( 'Media finalization failed:', error );
}
```

After:
```js
} catch ( error ) {
    console.warn( 'Media finalization failed:', error );
    dispatch.cancelItem(
        id,
        new UploadError( {
            code: ErrorCode.MEDIA_FINALIZE_ERROR,
            message: __( 'Could not finalize the upload.' ),
            file: item.file,
        } )
    );
    return;
}
```

A new error code `MEDIA_FINALIZE_ERROR` is added to `packages/upload-media/src/upload-error.ts`, and a corresponding user-facing message is registered in `packages/upload-media/src/error-messages.ts`. The `cancelItem` action already includes a guard that skips retries once an attachment ID exists, preventing infinite loops. No new public hooks, REST schema changes, or block.json fields are introduced; this operates entirely within the client-side upload queue.

## Contribution

Opened and merged by @ramonjd, with review and iteration from @andrewserong. The initial implementation focused solely on cancelling the queue item and surfacing the error. During review, @andrewserong suggested automatically deleting the orphaned attachment on failure to match existing sideload-failure behavior, but later reconsidered after weighing edge cases like gateway timeouts, potential cascading delete failures, and future needs for reprocessing. The team agreed to ship the narrower fix (cancel + error notice) and defer attachment cleanup to a separate effort.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
