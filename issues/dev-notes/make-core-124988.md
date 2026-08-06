# WordPress 7.1 Field Guide

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Milana Cap
- **Published:** 2026-08-05
- **Tags:** `Core`, `General`, `7.1`, `field guide`
- **Link:** [https://make.wordpress.org/core/2026/08/05/wordpress-7-1-field-guide/](https://make.wordpress.org/core/2026/08/05/wordpress-7-1-field-guide/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces a coordinated set of developer-facing changes that modernize media workflows, enforce an iframed editor environment, and expand extensibility through new APIs. Key additions include client-side media processing, standardized SVG icon registration, and new block supports for background gradients and minimum width. These shifts improve editor consistency and REST API capabilities but require plugin authors to audit cross-frame JavaScript interactions and update legacy dependency expectations.

## Impact

- **Plugin & theme developers:** Must audit JavaScript and CSS that previously relied on direct DOM access across the editor document boundary, as legacy metaboxes are now fully iframed. Plugins depending on jQuery UI 1.14.2 should run regression tests for styling or behavior shifts.
- **Block developers:** Can opt into `background.gradient` and `minWidth` controls via block metadata. `WP_Block_Type_Registry::register()` now includes context in `_doing_it_wrong()` messages, improving debugging.
- **REST API consumers:** Attachment responses expose size-aware encoding quality, the sideload endpoint validates dimensions, and a single sideloaded file can be registered under multiple image sizes. `WP_REST_Attachments_Controller::get_attachment_filesize()` now safely handles non-integer metadata.
- **Site owners & admins:** Media Library grid uses infinite scrolling by default (per-user opt-out available). The admin toolbar persists across editor screens during client-side navigation.
- **No action required** for most sites, though the enforced iframed editor and jQuery UI update are the primary areas requiring proactive compatibility checks.

## Technical details

The enforced iframed editor isolates legacy metaboxes within an iframe, breaking direct `window.parent` or cross-document DOM queries; developers must migrate to `postMessage` or the `wp.data` store. New block supports are registered via `block.json` metadata, changing the opt-in pattern from:
```json
{
  "supports": {
    "background": { "gradient": true },
    "minWidth": true
  }
}
```
The Abilities API (`wp_get_abilities()`) now accepts filters for registered abilities, exposes execution lifecycle hooks, and includes a unified public exposure flag with JSON Schema preparation for client compatibility. The SVG Icon API standardizes registration and rendering pipelines, replacing ad-hoc component handling. REST endpoint changes in `WP_REST_Attachments_Controller` validate dimensions on sideload, expose size-aware encoding quality, and allow multiple image size registrations per file. Additional behavioral shifts include `notify_post_author` gaining final authority over post author notifications, `WP_Theme_JSON::to_ruleset()` preventing implicit type coercion, and `get_file_data()` correctly parsing headers prefixed with `<?`.

## Contribution

The 7.1 release was coordinated through over 310 core Trac tickets and 630+ Gutenberg fixes, with release leads explicitly deferring or reverting several initiatives after testing and community feedback. Proposals to hide the Classic block from the inserter, upgrade to React 19, enable real-time collaborative editing, and ship an "On This Day" dashboard widget were all pulled from the final release to allow further validation. The enforced iframed editor and jQuery UI 1.14.2 update were finalized after extensive cross-browser regression testing, while the Abilities API and SVG Icon API matured through iterative Gutenberg plugin releases before core integration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
