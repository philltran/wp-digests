# #83197: Accordion Item: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`7380c04`](https://github.com/WordPress/gutenberg/commit/7380c044eb879f512e6113c9f082e3169695a5cd)
- **Discussion:** [#83197](https://github.com/WordPress/gutenberg/pull/83197) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Accordion Item block (`core/accordion-item`) now supports the `background.gradient` block support, enabling background gradients via the Background panel in both the block inspector and Global Styles. Previously the block only had the legacy `color.gradients` support, which did not expose the newer Background panel gradient controls. This brings the block in line with Accordion and Pullquote as part of the ongoing design-tools consistency effort (related to #43241).

## Impact

- **Site owners / theme developers using Global Styles:** The Background panel for Accordion Item now includes a gradient picker. No code changes required; the support is opt-in via `block.json` and applies automatically once the block is updated.
- **Plugin & theme developers:** No breaking change. The legacy `color.gradients` support is retained alongside the new `background.gradient`, so existing gradient assignments continue to work. No PHP or render-callback changes are needed.
- **No action required** for existing sites. The change is additive and only surfaces a new UI control.

## Technical details

The change is a single addition to `packages/block-library/src/accordion-item/block.json`:

```json
"supports": {
  "html": false,
  "background": {
    "gradient": true
  },
  "color": {
    "background": true,
    "gradients": true
  }
}
```

The new `background.gradient` key is added alongside the existing `color.gradients` key. The block's `save.js` already renders a wrapper element, and the PHP render callback only injects Interactivity API and ARIA attributes, so no server-side template change is needed. The core-blocks reference (`docs/reference-guides/core-blocks/README.md`) and the block's own `README.md` are regenerated to list the new support. This mirrors the pattern used in #79840 (Accordion) and #79841 (Pullquote).

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no notable design debate; the approach follows the established pattern from the Accordion and Pullquote gradient PRs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
