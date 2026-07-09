# #79104: Allow setting viewport tablet and mobile values in theme.json

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Package] Editor`, `[Package] Block editor`, `Needs Dev Note`, `[Type] Feature`, `[Feature] Design Tools`, `[Package] Private APIs`, `[Feature] Style States`
- **Merged:** [`e402aff`](https://github.com/WordPress/gutenberg/commit/e402aff3ceb90d14ad0c934408cd12752de34dc1)
- **Discussion:** [#79104](https://github.com/WordPress/gutenberg/pull/79104) · 13 comments · 3 reactions
- **Usefulness:** 4/5

## Summary

Adds support for configurable viewport breakpoints in `theme.json` via `settings.viewport.mobile` and `settings.viewport.tablet`, replacing previously hardcoded 480px/782px defaults. Theme and plugin authors can now define custom breakpoint widths using `px`, `em`, or `rem` units, directly controlling the media queries applied to responsive block styles, visibility states, and layout overrides in the block editor.

## Impact

- **Theme developers:** Can override default breakpoints by adding a `viewport` object under `settings`. Must ensure `mobile` is strictly less than `tablet` to avoid graceful degradation to a single breakpoint fallback.
- **Plugin & block developers:** Block supports generating responsive media queries (`block-visibility`, `layout`, `states`) now dynamically reference the configured viewport instead of hardcoded constants. Existing themes using default breakpoints require no migration.
- **Site owners / platform teams:** No immediate action required; legacy breakpoint behavior is preserved when no custom configuration is provided.

## Technical details

- Registers `viewport` in `WP_Theme_JSON_Gutenberg::VALID_SETTINGS` with `mobile` and `tablet` keys.
- Introduces a new public static method: `WP_Theme_JSON_Gutenberg::get_viewport_media_queries( $viewport_settings = null, $options = array() )`. This method sanitizes input, validates against `/^(?:\d+|\d*\.\d+)(?:px|em|rem)$/`, and returns an array of `@media` queries keyed by style-state names (`@mobile`, `@tablet`).
- Replaces direct iteration over `WP_Theme_JSON_Gutenberg::RESPONSIVE_BREAKPOINTS` in `lib/block-supports/block-visibility.php`, `layout.php`, and `states.php` with calls to the new method. Adds viewport config passthrough to block style variations.
- Sanitization logic (`sanitize_viewport_settings`) drops `tablet` if it is ≤ `mobile`, falling back to a single max-width query. Invalid values trigger an immediate return of defaults. The desktop breakpoint dynamically inherits the highest valid configured width when `include_desktop` is true.
- Updates `docs/how-to-guides/themes/global-settings-and-styles.md` and `theme-json-living.md` to document the schema, unit restrictions, and fallback behavior.

## Contribution

Opened by @tellthemachines to address #75707, merged as commit `e402aff`. Co-authored by @ramonjd, @talldan, @t-hamano, and @andrewserong. Review focused on edge cases around invalid or out-of-order breakpoints, ultimately settling on graceful single-breakpoint fallback rather than enforcing strict ordering. Legacy hardcoded breakpoints in core blocks were acknowledged during review but deferred to keep scope tight.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
