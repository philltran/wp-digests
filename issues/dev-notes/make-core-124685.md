# New Block Support in WordPress 7.1: Minimum Width

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aaron Robertshaw
- **Published:** 2026-07-26
- **Tags:** `General`, `7.1`, `blocks`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/26/new-block-support-in-wordpress-7-1-minimum-width/](https://make.wordpress.org/core/2026/07/26/new-block-support-in-wordpress-7-1-minimum-width/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 adds a `minWidth` dimension block support, allowing blocks to enforce a minimum width constraint natively. This fills a gap in the existing dimension controls by mirroring the `minHeight` support, applying the `min-width` CSS property to the block wrapper. Theme authors and block developers can now expose this constraint through the Dimensions panel without relying on custom CSS or block styles.

## Impact

- **Block & theme developers:** No immediate action required. Blocks that opt in via `block.json` or themes that enable it in `theme.json` will expose a new control in the Dimensions panel.
- **Site owners:** No action required. The feature is additive and disabled by default unless explicitly enabled in Global Styles or block settings.
- **Hosting & platform teams:** No migration or configuration changes needed. The change is fully backwards compatible.

## Technical details

The change registers `minWidth` under the existing `dimensions` block support. When a block declares `"supports": { "dimensions": { "minWidth": true } }` in `block.json`, the editor injects a “Minimum width” input into the Dimensions panel. The value is rendered as the `min-width` CSS property on the block’s wrapper element. The control respects theme-defined `dimensionSizes` presets, mapping selections to `--wp--preset--dimension--{slug}` custom properties. In `theme.json`, developers can enable the control globally via `settings.dimensions.minWidth` or apply a default value via `styles.dimensions.minWidth`. The UI visibility follows existing optional design tool rules: hidden by default in the block inspector unless enabled via `__experimentalDefaultControls`, but visible by default in Global Styles.

## Contribution

The record carries no discussion detail beyond the PR and issue links, and the feature was implemented as a straightforward additive extension to the existing `dimensions` support system.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
