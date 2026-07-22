# #80510: Dashboard Widgets: harden action `href` sanitization

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Type] Experimental`, `[Feature] Dashboard`, `[Package] Widget primitives`, `[Package] Widget Dashboard`
- **Merged:** [`918c8c6`](https://github.com/WordPress/gutenberg/commit/918c8c60f98cfdd0427f4d30615873dbad538089)
- **Discussion:** [#80510](https://github.com/WordPress/gutenberg/pull/80510) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Hardens sanitization for experimental dashboard widget action `href`s by rejecting unsafe protocols (`data:`, `javascript:`), resolving widget-local relative paths to plugin URLs at registration, and dropping unresolved relative non-admin paths to prevent `esc_url_raw()` from inventing `http://` URLs. The `download` attribute is now sanitized via `sanitize_file_name()`, and invalid actions trigger `_doing_it_wrong()` notices. This prevents potential XSS vectors and broken links in widget action definitions.

## Impact

- **Experimental dashboard widget developers:** Widget action `href`s are now strictly validated at registration. `data:` and `javascript:` URLs are silently dropped, and relative paths pointing to non-existent files are removed rather than rewritten. The `download` filename is sanitized to strip path traversal sequences.
- **Widget authors:** Local asset downloads must now exist in the widget's directory or use absolute `https://` URLs. Query strings on local filenames (e.g., `report.csv?v=2`) will not resolve as widget files.
- **No action required** for standard dashboard widgets, core plugins, or sites not using the experimental widget system.

## Technical details

- New function `gutenberg_resolve_widget_action_href( $href, $dir_name )` in `lib/experimental/dashboard-widgets/widget-types.php` handles path resolution:
  - Passes through absolute, scheme-relative, root-relative, and single-segment admin `.php` paths.
  - Drops paths containing `..` or relative non-admin files that do not exist under `widgets/{dir_name}/`.
  - Returns `''` for invalid paths, which causes `gutenberg_sanitize_widget_actions()` to drop the action and log a `_doing_it_wrong()` notice.
- `gutenberg_sanitize_widget_actions()` now accepts an optional `$dir_name` parameter, runs resolved `href`s through `esc_url_raw()`, and sanitizes the `download` attribute via `sanitize_file_name()`.
- `gutenberg_register_widget_types()` passes `$widget['dir_name']` to the sanitizer during registration.
- Documentation in `packages/widget-primitives/README.md` and `packages/widget-primitives/src/stories/actions.md` updated to reflect that `data:`/`javascript:` hrefs are rejected and local files are resolved at registration.
- Example stories migrated from inline `data:` URLs to static files (e.g., `goal-progress.csv`, `greeting.txt`).

## Contribution

Opened and merged by @retrofox with co-authorship from @chihsuan. The PR evolved from an initial help-link sanitization fix (#80409) into a broader action-URL hardening effort after review flagged the need to handle `data:`/`javascript:` protocols and local asset resolution. An earlier attempt to synthesize `rel` attributes on `@wordpress/ui` `Link` components was explicitly scoped out and closed in a follow-up (#80513). The final implementation prioritizes server-side validation at widget registration to prevent client-side injection or broken links.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
