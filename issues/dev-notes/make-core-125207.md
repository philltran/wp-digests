# Introducing name and informational tool tips in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Joe Dolson
- **Published:** 2026-08-03
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/08/03/introducing-name-and-informational-tool-tips-in-wordpress-7-1/](https://make.wordpress.org/core/2026/08/03/introducing-name-and-informational-tool-tips-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces two new PHP functions, `wp_get_tooltip()` and `wp_get_toggletip()`, to render accessible tooltips and informational toggles in the administration interface. These functions address accessibility gaps for icon-only controls and complex UI elements where persistent text labels or `aria-describedby` attributes are impractical. Core now uses them for post meta box controls and the login screen’s “Remember Me” checkbox, providing a standardized, accessible pattern for plugin and theme developers to adopt.

## Impact

- **Plugin & theme developers:** Can now use `wp_get_tooltip()` and `wp_get_toggletip()` to add accessible labels and help text to icon-only buttons or complex controls without writing custom ARIA markup.
- **Site owners & administrators:** Experience improved accessibility in core admin screens (post meta boxes, login screen) with visible names and expandable help text.
- **Hosting & platform teams:** No immediate configuration changes required; the `wp-tooltip` CSS is loaded globally, while JS is enqueued only where needed.
- **No action required** for existing sites unless you are building custom admin UIs that previously relied on non-standard tooltip implementations.

## Technical details

The change adds two functions to core: `wp_get_tooltip( string $content, array $args )` and `wp_get_toggletip( string $content, array $args )`. Both accept `$content` and an `$args` array supporting `id`, `button`, `label`, `close_label`, `icon`, and `class`. The functions output valid inline markup using `<span>` wrappers to prevent invalid nesting inside paragraphs. `wp_get_tooltip()` renders a hint popover using `popover="hint"`, while `wp_get_toggletip()` uses `popover="auto"` with `role="dialog"` and a close button. To use the feature outside core screens, enqueue the assets via `wp_enqueue_style( 'wp-tooltip' )` and `wp_enqueue_script( 'wp-tooltip' )`. The `WP_HTML_Tag_Processor` is used internally to attach required ARIA attributes to existing buttons passed via the `$button` argument.

## Contribution

The feature was developed to address accessibility gaps identified in ticket #51006, with implementation led by Joe Dolson and reviewed by @afercia, including contributions from @wongjn. The design prioritized inline `<span>` output to ensure safe insertion within any parent element, avoiding the invalid nesting that `<div>` or `<section>` would cause. The approach was finalized for WordPress 7.1 without significant public debate, focusing on a lightweight, standards-compliant popover implementation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
