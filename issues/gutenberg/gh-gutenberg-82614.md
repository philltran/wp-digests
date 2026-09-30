# #82614: Media: Stop injecting crossorigin attributes under Document-Isolation-Policy

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Block editor`, `Backport to WP Minor Release`
- **Merged:** [`265c3d2`](https://github.com/WordPress/gutenberg/commit/265c3d2a80a7299254895cf81e5233dfee73bdf1)
- **Discussion:** [#82614](https://github.com/WordPress/gutenberg/pull/82614) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Gutenberg's client-side media processing now sends the `Document-Isolation-Policy: isolate-and-credentialless` header directly and no longer injects `crossorigin="anonymous"` into cross-origin `<audio>`, `<video>`, `<source>`, `<script>` and `<link>` tags. The injection (a PHP output buffer rewriting the full page with `WP_HTML_Tag_Processor`, plus a JS MutationObserver) turned those loads into CORS requests, which fail for hosts without `Access-Control-Allow-Origin`, such as CDN-offloaded media. Under `isolate-and-credentialless`, cross-origin subresources load without credentials and need no attribute. The PR fixes Gutenberg issue #82691 and is tagged for backport to a WordPress minor release.

## Impact

**Site owners**
- Media and assets served from CDNs or other origins without CORS headers should load and play again in the editor and media library (audio/video previews previously failed with media error code 4).
- No action required.

**Plugin & theme developers**
- Third-party scripts and styles enqueued on `enqueue_block_editor_assets` no longer get `crossorigin="anonymous"` added.
- Code that reads canvas pixels from remote images or video must set `crossOrigin` on the element it creates, optionally via the `media.crossOrigin` filter. The Cover block's dominant color and the image editor already do this.
- Removed PHP functions: `gutenberg_start_cross_origin_isolation_output_buffer()` and `gutenberg_add_crossorigin_attributes()`. The replacement for the former is `gutenberg_send_document_isolation_policy_header()`. The `gutenberg_use_document_isolation_policy` filter is unchanged. Anything calling the removed functions will fatal.
- `gutenberg_update_media_template_crossorigin_attributes()` is renamed to `gutenberg_remove_media_template_crossorigin_attributes()`.
- The `cross-origin-isolation` hook in `@wordpress/block-editor` is removed (it had no exports, only side effects).
- Per the updated docs, resources that depend on signed cookies or a logged-in session are the one case that can break under `credentialless`.

**Hosting & platform**
- The header is still only sent on the post editor, site editor and widgets screens, and is skipped for `action` parameters other than `edit`.
- The PHP output buffer over the whole admin page is gone.

**Core / Safari & Firefox**
- Core backport is tracked in wordpress-develop PR #13446 / Trac #65930. The PR text says Safari keeps its injection through the Client-Side Media Everywhere plugin.

## Technical details

**PHP (`lib/media/load.php`)**
- `gutenberg_send_document_isolation_policy_header(): bool` replaces the output-buffer starter. After the existing Chromium-version and `gutenberg_use_document_isolation_policy` filter checks, it calls `header( 'Document-Isolation-Policy: isolate-and-credentialless' )` directly and returns `true`, or `false` if skipped. `gutenberg_set_up_cross_origin_isolation()` calls it.
- `gutenberg_add_crossorigin_attributes()` (the `WP_HTML_Tag_Processor` walker with bookmarks for `SOURCE` -> parent `AUDIO`/`VIDEO`) is deleted.
- `gutenberg_override_media_templates()` now returns early unless `wp_add_crossorigin_attributes()` exists and `wp_send_document_isolation_policy_header()` does not. That limits it to WordPress 7.1, the one release whose `wp_print_media_templates()` injects the attribute itself.
- The template filter now removes `crossorigin="anonymous"` from `AUDIO`, `IMG` and `VIDEO` tags inside the `<script type="text/html">` Backbone templates, where it previously added it.

**JS (`packages/block-editor`)**
- `src/hooks/cross-origin-isolation.js` (the `window.crossOriginIsolated`-gated MutationObserver) and its jsdom test are deleted, and the import is removed from `src/hooks/index.js`. The changelog notes about -302 B on `build/scripts/block-editor/index.min.js`.

**Docs**
- `client-side-media-architecture.md`, `how-to-guides/client-side-media.md` and `lib/media/docs/client-side-media-docs.md` are rewritten to explain why no attribute is needed. A `backport-changelog/7.2/13446.md` entry is added.

```php
// Before
ob_start( fn( $o ) => header( 'Document-Isolation-Policy: isolate-and-credentialless' ) ?? gutenberg_add_crossorigin_attributes( $o ) );

// After
header( 'Document-Isolation-Policy: isolate-and-credentialless' );
```

(Before snippet is a condensed paraphrase of the removed closure, which sent the header inside the buffer callback.)

## Contribution

Authored by @adamsilverstein, who wrote the code and description with Claude Code and ran the testing checklist through Claude-driven browser automation across Chromium, Firefox and WebKit. The injection dated from the earlier `require-corp` COEP experiment and was carried over unchanged into the DIP switch. An earlier fix (#76618) excluded only `<img>` after CDN images broke, and whether the remaining tags should go too was left unanswered until this PR. @swissspidy asked to confirm the attribute is only needed under COEP/COOP and not DIP, and approved on that basis. Adam later added a nuance that the attribute only matters under a `require-corp` policy, not `credentialless`. Adam's report notes that Safari itself and a side-by-side trunk run were not covered. Adam marked it for backport to the next core minor (7.1.2).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
