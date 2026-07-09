# #74913: Media: Add hooks and extension points for client-side media processing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Status] In Progress`, `[Package] Editor`, `[Package] Block editor`, `Needs Dev Note`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`e1f2729`](https://github.com/WordPress/gutenberg/commit/e1f27295fa80a8abae92ea95e3218babc21aa85d)
- **Discussion:** [#74913](https://github.com/WordPress/gutenberg/pull/74913) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds extension points for client-side media processing to ensure server-side attachment hooks continue firing after browser-based image resizing and thumbnail generation. Introduces a new REST endpoint (`POST /wp/v2/media/{id}/finalize`) that re-triggers `wp_generate_attachment_metadata` with `$context === 'update'`, alongside a new `OperationType.Finalize` pipeline operation in the JS media upload store. This prevents plugins relying on server-side post-processing (watermarking, CDN sync, custom sizes) from breaking when client-side processing is enabled.

## Impact

- **Plugin Developers:** Media-handling plugins using `add_filter( 'wp_generate_attachment_metadata' )` should check `$context === 'update'` to ensure their logic executes after the new client-side pipeline completes.
- **Block/Editor Plugin Authors:** The internal upload-media package now uses an injected callback pattern instead of direct REST calls; custom editor integrations may need alignment with the new settings flow.
- **Site Owners & Platform Teams:** No immediate action required. The PHP changes are scoped for backporting alongside the client-side media experiment landing in Core.

## Technical details

- **PHP:** Adds a `finalize` route, `finalize_item()`, and `finalize_item_permissions_check()` to `WP_REST_Attachments_Controller`. The handler invokes attachment metadata generation while passing `$context = 'update'`.
- **JS Store & Pipeline:** Introduces `OperationType.Finalize` in `packages/upload-media/src/store/types.ts` and a new store setting `imageQuality` (0–1) for controlling resize/crop fidelity.
- **Architecture Shift:** `finalizeItem()` in `private-actions.ts` refactors from direct `apiFetch` calls to consuming an injected `finalizeUpload` callback. The dependency is wired through `packages/editor/src/components/provider/use-block-editor-settings.js` and passed via `packages/block-editor/src/components/provider/use-media-upload-settings.js`, removing `@wordpress/api-fetch` as a hard dependency for the upload-media package.
- **Code Example (PHP hook update):**
  ```php
  add_filter( 'wp_generate_attachment_metadata', function( $metadata, $attachment_id, $context ) {
      if ( 'update' === $context ) {
          // Trigger after client-side media pipeline completes
      }
      return $metadata;
  }, 10, 3 );
  ```

## Contribution

Opened by `@adamsilverstein` to resolve #74358 and support the ongoing client-side media processing experiment. Co-authored with `@andrewserong`, `@swissspidy`, and `@westonruter`. The PR establishes a dependency-injection pattern for the finalize step to keep the upload-media package agnostic of REST implementation details, and defines a clear backport scope (PHP controller changes only) targeting Core integration after the client-side experiment ships.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
