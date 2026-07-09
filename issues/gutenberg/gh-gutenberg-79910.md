# #79910: Editor: Preload the view config form request for the DataForm inspector

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Enhancement`, `[Package] Edit Post`
- **Merged:** [`6ffa192`](https://github.com/WordPress/gutenberg/commit/6ffa1929f13088b16270854db5c72edf7ebb2bc1)
- **Discussion:** [#79910](https://github.com/WordPress/gutenberg/pull/79910) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This change optimizes the DataForm-based document inspector by preloading the `/wp/v2/view-config` form configuration request during editor kickoff. When the `gutenberg-dataform-inspector` experiment is active, the view config is now served directly from the HTTP preload cache rather than triggering a synchronous network round-trip on sidebar mount. This eliminates the UI flash where the document summary rendered as an empty form before flashing in once the response arrived.

## Impact

- **Plugin & theme developers / editor experimenter**: No breaking changes or new public APIs. The `gutenberg-dataform-inspector` experiment now expects `/wp/v2/view-config?kind=postType&...&_fields=form` to be consumed from the preload cache during kickoff. If you are building custom inspector UIs or overriding resolver logic under this flag, ensure your initialization matches the preloaded `viewConfig` shape to prevent cache misses.
- **Site owners & agencies**: No action required. The change is strictly gated behind an explicit experiment flag and improves editor performance without altering default block editor behavior or REST API contracts.

## Technical details

- Adds `lib/experimental/dataform-inspector-preload.php`, which hooks into `block_editor_rest_api_preload_paths` to append `/wp/v2/view-config?kind=postType&name={$context->post->post_type}&_fields=form` for `core/edit-post` contexts where a post exists.
- In `packages/edit-post/src/index.js`, the `preloadResolutions()` kickoff now conditionally invokes `unlock( core ).getViewConfig( 'postType', postType, { fields: 'form' } )` when `window?.__experimentalDataformInspector` is truthy. This aligns the client-side resolver arguments with the PHP preload middleware's normalized path (automatically stripping `_locale=user` as per api-fetch preloading behavior).
- `lib/load.php` conditionally requires the new PHP file only when `gutenberg_is_experiment_enabled( 'gutenberg-dataform-inspector' )` returns true, preventing unnecessary page weight in standard editor loads.
- E2E tests in `test/e2e/specs/preload/post-editor.spec.js` assert that all preloaded paths are consumed during kickoff and verify zero network requests hit `/wp/v2/view-config`.

## Contribution

Opened and merged by @jorgefilipecosta with co-authorship from @ntsekouras (merged as `6ffa192`). The PR focused on synchronizing the frontend resolver args with the PHP-generated preload path to guarantee cache hits and eliminate synchronous fetching. An E2E test suite was added to catch preload drift or dropped resolvers, validating both cache consumption and network suppression.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
