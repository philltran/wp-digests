# #83563: Media & Text: Lower the specificity of the content padding

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Media & Text`
- **Merged:** [`5666ca1`](https://github.com/WordPress/gutenberg/commit/5666ca1db73e2efc70007bfab5b17f412bdf0375)
- **Discussion:** [#83563](https://github.com/WordPress/gutenberg/pull/83563) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Media & Text block's default content padding (`padding: 0 8%`) is now emitted through a `:where()`-wrapped selector, giving it zero specificity. Themes can override that padding via `theme.json` custom CSS or their own stylesheets without resorting to `!important`. The default rendered appearance is unchanged, and no markup is altered, so there are no deprecation or block-validation concerns.

## Impact

- **Theme developers:** The long-standing `!important` workaround for the 8% side padding is no longer needed. A plain rule targeting `.wp-block-media-text__content` (or the `theme.json` `css` key under `styles.blocks.core/media-text`) now wins by specificity.
- **Plugin & block developers:** No action required. No public API, hook, or `block.json` field changed.
- **Site owners / hosting:** No visible change unless a theme or custom CSS was already fighting the padding with `!important`; those rules can now be simplified.
- **Headless / REST consumers:** No effect; the change is front-end CSS only.

## Technical details

In `packages/block-library/src/media-text/style.scss`, the `padding: 0 8% 0 8%;` declaration was removed from the existing `.wp-block-media-text > .wp-block-media-text__content` rule and re-emitted as a separate, zero-specificity rule:

```scss
/* Before (inside the higher-specificity rule) */
.wp-block-media-text > .wp-block-media-text__content {
  /* … */
  padding: 0 8% 0 8%;
  word-break: break-word;
}

/* After */
.wp-block-media-text > .wp-block-media-text__content {
  /* … */
  word-break: break-word;
}

:where(.wp-block-media-text > .wp-block-media-text__content) {
  padding: 0 8%;
}
```

Because `:where()` contributes zero to the specificity calculation, any theme rule that matches the same element (e.g. `.wp-block-media-text__content { padding: 0; }`) now outranks the block's default. The compiled output appears in `build/styles/block-library/media-text/style.css` (and the RTL variant), adding roughly 5–11 bytes per file. No `block.json`, REST schema, or database changes accompany the PR.

## Contribution

Opened by @im3dabasia as part of the broader #28556 effort to make Media & Text spacing more flexible. The PR explicitly positions itself as a stopgap: the fuller gap-support approach in #67247 is blocked on an unresolved design question (#67208) about where the gap applies, and it would alter saved markup. Co-authored with @mukeshpanchal27 and @ciampo. The discussion was minimal (2 comments, 0 reactions), with no notable design debate recorded in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
