# #80845: Boot: Adjust specificity of the image reset styles so components can size their own images

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Bug`, `Backported to WP Core`, `[Package] Boot`
- **Merged:** [`0d39277`](https://github.com/WordPress/gutenberg/commit/0d39277519526c6afc2d8817a70121b515d8a686)
- **Discussion:** [#80845](https://github.com/WordPress/gutenberg/pull/80845) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/boot` package adjusted the CSS specificity of its default `img` reset so that child components can reliably override image dimensions. Previously, the reset compiled to a nested selector that outweighted component-level sizing rules, causing layout jumps in interfaces like the Font Library. Wrapping the selector in `:where()` strips its specificity weight while preserving the safety-net behavior for unconstrained images.

## Impact

- **Plugin & theme developers using `@wordpress/boot`**: Components that set explicit image heights or widths inside the boot layout will now correctly apply their styles instead of being overridden by the global reset.
- **WordPress admin UI consumers**: Fixes visual inconsistencies and layout shifts in the Font Library and other boot-layout-dependent pages.
- No breaking changes, deprecations, or migration steps required; the change is purely a CSS specificity adjustment.

## Technical details

The change modifies `packages/boot/src/style.scss`. The original nested rule compiled to a two-level selector (e.g., `.boot-layout-container .boot-layout img`), which contributed extra specificity to the cascade. The diff flattens the selector and wraps it in `:where(.boot-layout) img`, reducing its specificity to `(0,1,0)` while keeping the `max-width: 100%; height: auto;` constraints. This allows any descendant component (e.g., `.font-library__font-variant_demo-image`) to override the height with a more specific rule or standard cascade priority. The change also notes compatibility with the ongoing SCSS module conversion in #80359, where the hashed class will still behave identically inside `:where()`.

## Contribution

Opened by @juanfra to resolve layout jumps in the Font Library, the PR was reviewed by @aduth and @simison. Discussion centered on whether the reset was still necessary and how it would interact with the ongoing SCSS module migration (#80359); the author confirmed the fix remains valid post-migration and debated removing the rule entirely versus keeping it as a safety net for uncontrolled content. @simison approved it as a bug fix and requested inclusion in WordPress 7.1, leading to a cherry-pick to the `wp/7.1` branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
