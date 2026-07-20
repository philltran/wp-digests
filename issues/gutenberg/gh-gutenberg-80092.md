# #80092: Cover: allow restricting video embed providers

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Cover`
- **Merged:** [`06f9fb8`](https://github.com/WordPress/gutenberg/commit/06f9fb85bf6ff40852ce526f542c6e623b598e97)
- **Discussion:** [#80092](https://github.com/WordPress/gutenberg/pull/80092) · 7 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The Cover block now supports an `allowedVideoProviders` attribute that lets developers restrict or disable background video embedding from URLs. Previously, the block relied on a hard-coded provider list with no way to curate it. This enhancement allows filtering the provider list via block markup or the `register_block_type_args` filter, automatically hiding the "Embed video from URL" UI option when no providers remain allowed.

## Impact

- Plugin & theme developers: Can restrict or disable background video embedding for the `core/cover` block by setting the `allowedVideoProviders` attribute in block markup or filtering it via `register_block_type_args`.
- Site owners & editors: Will see the "Embed video from URL" menu item hidden or restricted based on the configured allowed providers.
- No breaking changes: Existing Cover blocks with embedded videos continue to render and function normally; only new URL input in the editor is restricted.
- No immediate migration required, but sites relying on unrestricted video embedding should audit their block templates or global filters if they want to enforce restrictions.

## Technical details

- Adds `allowedVideoProviders` (type: `array`, default: `["youtube","vimeo","videopress","animoto","tiktok","wordpress-tv"]`) to `packages/block-library/src/cover/block.json`.
- Updates `packages/block-library/src/cover/edit/block-controls.js` to conditionally render the "Embed video from URL" `MenuItem` inside `MediaReplaceFlow` based on `hasAllowedVideoProviders`.
- Modifies `packages/block-library/src/cover/embed-video-utils.js`: renames `VIDEO_PROVIDERS` to `DEFAULT_VIDEO_PROVIDERS`, introduces `getAllowedVideoProviders()` to filter against the hard-coded list, and updates `isValidVideoEmbedUrl()`, `getVideoEmbedProvider()`, and `findVideoEmbedProvider()` to accept an `allowedProviders` parameter. Unknown slugs are safely ignored.
- Passes `allowedVideoProviders` to `EmbedVideoUrlInput` in `edit/block-controls.js`, which forwards it to `isValidVideoEmbedUrl()` for validation.
- Example filter to restrict to Vimeo only:
  ```php
  add_filter( 'register_block_type_args', function ( $args, $block_type ) {
      if ( 'core/cover' === $block_type && isset( $args['attributes']['allowedVideoProviders']['default'] ) ) {
          $args['attributes']['allowedVideoProviders']['default'] = [ 'vimeo' ];
      }
      return $args;
  }, 10, 2 );
  ```
- Minor UI refactor: replaces `__experimentalVStack` with `Stack` in `embed-video-url-input.js`.

## Contribution

Opened and merged by @t-hamano with review and co-authorship from @Mamaduka and @annezazu. The PR resolves #75069 by implementing the restriction capability. During review, @Mamaduka noted that adding custom providers would require provider-specific query parameter filters, but @t-hamano determined that was out of scope for now, and the group agreed to close the parent issue as resolved for the current use case.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
