# #78332: Icons: Add PHP method(s) for rendering inline SVG icons from the registry

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Package] Block library`, `Needs Dev Note`, `[Type] Feature`, `[Package] Icons`, `[Feature] Icons`
- **Merged:** [`e7c373b`](https://github.com/WordPress/gutenberg/commit/e7c373b6a018b629088964c032c22ec3c621ef3f)
- **Discussion:** [#78332](https://github.com/WordPress/gutenberg/pull/78332) · 22 comments · 3 reactions
- **Usefulness:** 4/5

## Summary

Introduces `wp_get_icon()` as a new PHP helper that retrieves inline SVG markup from the WordPress Icons Registry. It provides a server-side equivalent to the React `<Icon>` component, ensuring consistent sizing, CSS classes, and accessibility attributes across PHP templates and block contexts. This eliminates the need for themes and plugins to hand-roll icon rendering or manage external asset paths.

## Impact

- **Plugin & theme developers**: Gain an official, opt-in API to render icons in PHP without duplicating logic or managing file paths.
- **Block authors**: The `core/icon` block's server renderer (`render_block_core_icon()`) is refactored to delegate core SVG/ARIA generation to this helper, aligning front-end and back-end output.
- **Action required**: None for existing installations. Developers integrating the function should verify that the default behavior matches their accessibility requirements (decorative vs. meaningful icons).

## Technical details

- Adds `wp_get_icon( string $name, array $args = [] ): string` to the Icons package.
- Resolves `$icon_name` via `WP_Icons_Registry::get_instance()->get_registered_icon()`, returning an empty string on miss.
- Injects presentation attributes using `WP_HTML_Tag_Processor`: sets `width`/`height` from the `size` argument (defaults to `24`), and applies space-separated `$args['class']` values.
- Handles ARIA semantics: when `$args['label']` is provided, it attaches `role="img"` and `aria-label`; otherwise it sets `aria-hidden="true"` and `focusable="false"`.
- Filters the final markup via `apply_filters( 'wp_icon_html', $html, $name )`.
- Refactors `render_block_core_icon()` to call `wp_get_icon()` for base SVG/ARIA generation, while preserving block-level styling (colors, spacing, dimensions) through the style engine.

## Contribution

Opened and merged by @jasmussen as part of the ongoing Icons package effort. Reviewer @t-hamano approved the approach of combining the registry with `WP_HTML_Tag_Processor` for attribute injection, noting it eliminates redundant generation logic. The feature was finalized in Gutenberg and backported to WordPress core via `wordpress-develop#12010`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
