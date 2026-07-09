# #79409: Image: sideload external images on the server when uploading to the library

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Status] In Progress`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Feature] Client Side Media`
- **Merged:** [`bdde544`](https://github.com/WordPress/gutenberg/commit/bdde544f4aaba2a282b5b548dd53bb2fc8d2bdf8)
- **Discussion:** [#79409](https://github.com/WordPress/gutenberg/pull/79409) · 17 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Extends the POST /wp/v2/media REST endpoint to accept an optional url parameter, shifting external image uploads from client-side browser fetches to server-side sideloading via media_handle_sideload(). This change fixes silent CORS failures in cross-origin isolated editors (client-side media processing) by letting WordPress core handle the remote download, eliminating the need for permissive hosting headers. The previously used client-side fetch helper is removed as it is no longer necessary.

## Impact

- **Plugin & theme developers**: May leverage the new `mediaSideloadFromUrl` editor setting to handle external media uploads without implementing browser-side blob fetching. If your code relies on the internal `fetchMedia()` utility or expects client-side fetch behavior for external URLs, it has been removed.
- **Site owners & editors**: External image uploads via "Upload to Media Library" or the pre-publish panel now reliably succeed regardless of the remote host's CORS policy.
- **Hosting & platform teams**: No immediate configuration changes required, though server-side HTTP requests will now originate directly from the REST endpoint when a `url` is provided.
- No action required for standard themes/plugins not interacting with external media upload flows.

## Technical details

- `lib/media/class-gutenberg-rest-attachments-controller.php` registers a `url` argument on the creatable `/wp/v2/media` route. When present, `create_item()` routes to a new private `create_item_from_url()` method.
- The server-side flow uses `download_url()` to fetch the remote file, validates it against SSRF risks via `wp_http_validate_url()`, and sideloads it with `media_handle_sideload()`. It explicitly fires `rest_after_insert_attachment` for parity and respects existing sub-size scaling filters.
- JavaScript side: The new `mediaSideloadFromUrlKey` is registered in `block-editor/src/private-apis.js` and `private-keys.js`. Both `blocks-library/src/image/image.js` and `editor/src/components/post-publish-panel/maybe-upload-media.js` now consume this setting instead of the deleted `fetchMedia()` utility.
- Before/after pattern:
  ```javascript
  // Old (removed)
  const mediaUpload = getSettings().mediaUpload;
  mediaUpload({ filesList: [ externalBlob ], ... });

  // New
  const mediaSideloadFromUrl = getSettings()[ mediaSideloadFromUrlKey ];
  mediaSideloadFromUrl({ url, onSuccess( img ) { onSelectImage( img ); }, onError() { ... } });
  ```

## Contribution

Opened by @adamsilverstein and merged following reviews from @ramonjd, @Mamaduka, @andrewserong, @westonruter, and @swissspidy. The discussion centered on ensuring server-side sideloading remains the preferred path to avoid wasteful double-fetches and resolve cross-origin isolation breakages (#79407). An alternative to disable auto-sideload entirely was debated but ultimately rejected in favor of preserving the functionality while moving it safely to the server. All associated e2e tests were re-enabled once this server-side path landed.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
