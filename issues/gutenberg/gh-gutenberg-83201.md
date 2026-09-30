# #83201: Accordion Panel: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`579bce8`](https://github.com/WordPress/gutenberg/commit/579bce8a1fbba83c4bbb2f3387759d5e6defdffb)
- **Discussion:** [#83201](https://github.com/WordPress/gutenberg/pull/83201) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Accordion Panel block (`core/accordion-panel`) now supports background image, background size, and gradient via the `background` block supports key. Previously the block only supported color (including legacy `color.gradients`), border, layout, shadow, spacing, and typography. This change brings the panel in line with other core blocks as part of the ongoing design-tools consistency effort tracked in #43241.

## Impact

- **Theme & site builders (Global Styles):** The Accordion Panel now exposes Background → Image, Size, and Gradient controls in the Global Styles panel (Blocks → Accordion Panel → Background). No code changes needed; the controls appear automatically once the block supports are registered.
- **Block-level overrides:** The same three controls are available in the block inspector for individual Accordion Panel instances, with block-level values overriding Global Styles values.
- **Responsive styles:** All three supports respect the responsive (mobile/desktop) states in Global Styles and the block inspector.
- **No breaking changes.** The legacy `color.gradients` support remains alongside the new `background.gradient` (mirroring the pattern established for the parent Accordion block in #79840). No PHP, REST, or database changes are required.
- **No action required** for existing sites; the new supports are additive and default to unset.

## Technical details

The change is confined to the block's `block.json` and its generated documentation. In `packages/block-library/src/accordion-panel/block.json`, a new `background` object is added to the `supports` key:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

Because `core/accordion-panel` is a static block, these supports are serialized onto the wrapper element by `useBlockProps.save()` at render time — no PHP render callback or `render.php` change is needed. The gradient layers over the background image (per the stacking behavior introduced in #75859), so both are visible simultaneously.

The legacy `color.gradients: true` entry is retained in the `color` supports object alongside the new `background.gradient`, matching the dual-support pattern used by the parent `core/accordion` block (#79840).

Two documentation files are regenerated to reflect the new supports:
- `docs/reference-guides/core-blocks/README.md` — the Accordion Panel supports line now lists `background (backgroundImage, backgroundSize, gradient)`.
- `packages/block-library/src/accordion-panel/README.md` — a new `background` section documents the three sub-keys.

## Contribution

Opened by @aaronrobertshaw and co-authored with @ramonjd. The PR body notes the implementation was produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions), with no recorded design debate or rejected alternatives. It was merged as commit `579bce8`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
