# New Block Support in WordPress 7.1: Background Gradient (background.gradient)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aaron Robertshaw
- **Published:** 2026-07-26
- **Tags:** `General`, `7.1`, `blocks`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/26/new-block-support-in-wordpress-7-1-background-gradient-background-gradient/](https://make.wordpress.org/core/2026/07/26/new-block-support-in-wordpress-7-1-background-gradient-background-gradient/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces a new `background.gradient` block support that allows gradients to render alongside background images on the same block. Previously, gradients applied via `color.gradient` used the `background` shorthand, which reset and overrode any `background-image`. The new support stores values at `style.background.gradient` and renders via the `background-image` longhand property, enabling comma-separated gradient and image declarations without conflict.

## Impact

- **Plugin & theme developers:** Blocks that previously relied on `color.gradient` are unaffected, but developers can now opt into the new support in `block.json` to combine gradients with background images. No migration is required for existing blocks.
- **Site owners & editors:** The gradient control moves to the Background panel when `background.gradient` is enabled, and the Color panel gradient tab is suppressed to prevent duplicate controls.
- **Hosting & platform teams:** No action required. The change is additive and backwards compatible.
- **Headless & REST consumers:** The block attribute structure remains unchanged; styling is handled server-side via the style engine and injected as inline CSS.

## Technical details

The change registers a new block support key, `background.gradient`, which stores its value at `style.background.gradient` instead of `style.color.gradient`. In the style engine, the gradient is merged with any existing `background-image` into a single comma-separated `background-image` declaration (e.g., `background-image: linear-gradient(...), url(...);`). This avoids the `background` shorthand reset that previously caused conflicts. The `safecss_filter_attr()` function is updated to permit mixed gradient and `url()` values in `background-image`, removing the need for custom sanitization filters. Blocks opt in via `block.json`:
```json
{
  "supports": {
    "background": {
      "backgroundImage": true,
      "gradient": true
    }
  }
}
```
Core blocks `core/group`, `core/accordion`, `core/pullquote`, `core/post-content`, and `core/quote` enable this support in 7.1. Theme-level defaults can be set via `theme.json` under `styles.background.gradient`.

## Contribution

The record carries no discussion detail beyond the linked PR and issue, and the author explicitly deferred any broader migration from `color.gradient` to `background.gradient` to a future release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
