# #77036: Deduplicate client-side image sizes with matching dimensions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Status] In Progress`, `[Feature] Client Side Media`
- **Merged:** [`70387ef`](https://github.com/WordPress/gutenberg/commit/70387ef2176be24a53ef0b66f335689bdd5c19bf)
- **Discussion:** [#77036](https://github.com/WordPress/gutenberg/pull/77036) · 6 comments · 0 reactions

## Summary

Client-side image uploads now deduplicate physical files for theme sizes that match built-in dimensions (e.g., a custom `large` size matching `medium_large`). This prevents the creation of duplicate `-1` suffixed files during sideloading, aligning client-side behavior with server-side handling.

## Impact

- Theme authors & developers using client-side media: No action required; deduplication is handled automatically during upload.
- Code relying on `wp_get_attachment_metadata`: Multiple registered size keys (e.g., `large`, `medium_large`) will now point to the same physical file when their dimensions intersect, matching server-side behavior.
- Hosting/Platform: Reduces storage overhead caused by generating redundant client-side renditions for overlapping dimension sets.

## Technical details

- **Client-side (`packages/upload-media/src/store/private-actions.ts`)**: The sideload loop now groups `sizesToGenerate` by effective dimensions (width, height, crop) using a `Map`. Instead of issuing separate requests per size name, it aggregates matching names into a single array and sends one request.
- **Client-side (`packages/upload-media/src/store/types.ts`)**: Updated the type definition for `image_size` to accept `string | string[]`, reflecting the new capability to batch multiple size names.
- **Server-side (`lib/media/class-gutenberg-rest-attachments-controller.php`)**: The REST endpoint's `image_size` parameter schema was updated to accept `oneOf` string or array values. When an array is passed, the controller iterates over all provided size names and attaches them to the single uploaded file's metadata.

## Contribution

Merged in PR #77036 by @adamsilverstein with review from @swissspidy. The author noted a planned core backport (Trac ticket #65330) to be applied once client-side media is fully restored to WordPress Core. A test verifying this deduplication behavior was added to `packages/upload-media/src/store/test/actions.ts`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
