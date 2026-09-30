# #83252: Tabs: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Tabs`
- **Merged:** [`982d55a`](https://github.com/WordPress/gutenberg/commit/982d55a09a96e7942804af15978aafa25f222472)
- **Discussion:** [#83252](https://github.com/WordPress/gutenberg/pull/83252) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block (`core/tabs`) now declares `background.backgroundImage`, `background.backgroundSize`, and `background.gradient` in its `block.json` supports, enabling background image, size, and gradient controls in both the block inspector and Global Styles. This closes a gap where the block already supported color, layout, shadow, spacing, and typography but lacked the Background panel's image and gradient controls, as part of the broader design-tools consistency effort (#43241). No PHP or render-callback changes were needed because the existing `WP_HTML_Tag_Processor`-based render path preserves the serialized `class` and `style` attributes from `save.js`.

## Impact

- **Theme & site-editor developers:** The Tabs block now appears with Background image, size, and gradient controls in Global Styles (Blocks → Tabs → Background). If your theme or plugin reads `block.json` supports programmatically, the new `background` key is present on `core/tabs`.
- **Plugin developers:** No breaking change. The legacy `color.gradients` support remains alongside the new `background.gradient`; both are active. No migration or code change is required.
- **Site owners / editors:** New controls are available in the block inspector and Site Editor for the Tabs block. No action required.
- **Headless / REST consumers:** No REST schema or attribute changes. The new supports are serialized into the block's `class` and `style` attributes in the saved markup, so any consumer reading rendered HTML will see the new CSS properties.

## Technical details

The diff adds a `background` object to the `supports` key in `packages/block-library/src/tabs/block.json`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true
  }
}
```

The `__experimentalDefaultControls` entry enables the background-image control by default in the block inspector (not just via Global Styles). The Tabs wrapper element is produced in `save.js`, so the standard block-supports serialization pipeline emits the appropriate `class` and `style` attributes on the wrapper. The PHP render callback only appends Interactivity API attributes via `WP_HTML_Tag_Processor` and does not strip or overwrite those serialized attributes, so no `render.php` or `render_callback` change was necessary.

The gradient layers over the background image (per the behavior established in #75859), so both are visible simultaneously. The pre-existing `color.gradients` support is retained alongside `background.gradient` for backward compatibility.

Two documentation files are regenerated to list the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/tabs/README.md`.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR notes it was implemented, built, and screenshotted by a Claude Code agent from a predefined task. The discussion thread contains only automated bot comments (performance benchmarks and a flaky-test report); no human review comments or design debate are visible in the record.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
