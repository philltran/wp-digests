# #83363: Tab Panel: Add border support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`7ead797`](https://github.com/WordPress/gutenberg/commit/7ead797f385d0037efb602473f919fe7a4d0af36)
- **Discussion:** [#83363](https://github.com/WordPress/gutenberg/pull/83363) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panel block (`core/tab-panel`) now supports border colour, radius, style, and width via the `__experimentalBorder` supports key in its `block.json`. This closes a gap where the block already supported background, colour, shadow, spacing, and typography but had no border control, as part of the broader design-tools consistency effort (related to #43241). A companion CSS rule prevents closed panels—which remain in the layout via `hidden="until-found"`—from rendering a visible border strip beneath the open panel.

## Impact

- **Site owners / editors:** Tab Panel blocks can now be styled with borders through Global Styles (Blocks → Tab Panel → Border) or the block inspector, including per-breakpoint (mobile) values. No action required to adopt; the feature is opt-in via the existing design tools UI.
- **Plugin & theme developers:** No code changes needed. The `__experimentalBorder` key follows the same pattern used by other core blocks. No PHP render change is required because the saved `<section>` wrapper already carries the generated styles.
- **Headless / REST consumers:** No schema or route changes. The `__experimental` prefix means the key is excluded from generated block documentation.
- **No breaking changes, deprecations, or migrations.**

## Technical details

Two files change:

**`packages/block-library/src/tab-panel/block.json`** — a new `__experimentalBorder` object is added to the `supports` key, alongside the existing `__experimentalFontFamily` and other supports:

```json
"__experimentalBorder": {
  "color": true,
  "radius": true,
  "style": true,
  "width": true
}
```

**`packages/block-library/src/tab-panel/style.scss`** — the existing `&[hidden]` rule (which already zeroed `box-shadow`, `margin-block-start`, and `padding`) gains `border: none !important` so that closed panels do not paint a border strip. The comment is updated to mention borders explicitly.

Because the block's render output wraps each panel in a `<section>` element that receives the generated inline styles, no PHP template change is needed. The docs generators skip `__experimental`-prefixed keys, so the generated block reference docs are unchanged. Bundle impact is +52 B total (mostly the four new `block.json` booleans and the one-line SCSS addition).

## Contribution

Opened by @aaronrobertshaw and reviewed/rebased by @ramonjd. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The only review friction was a rebase conflict on generated README files, which @aaronrobertshaw described as a recurring pattern across design-tool adoption PRs and shared a script to automate the resolution. No design debate or alternative approaches are recorded in the four comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
