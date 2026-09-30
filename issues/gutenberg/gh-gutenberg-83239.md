# #83239: Preformatted: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Preformatted`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`ef553c8`](https://github.com/WordPress/gutenberg/commit/ef553c8a9bbbaeee180dfb66bb228dd56b6a96a5)
- **Discussion:** [#83239](https://github.com/WordPress/gutenberg/pull/83239) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Preformatted block (`core/preformatted`) now supports background image, background size, and gradient via the Background panel in Global Styles and the block inspector. Previously the block supported color (including legacy `color.gradients`), border, shadow, spacing, and typography, but lacked the Background panel's image, size, and gradient controls. This change brings the block in line with other core blocks as part of the design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / editors:** Can now set a background image (with `Cover`, `Contain`, etc. sizing) and a gradient on Preformatted blocks through Global Styles (Blocks → Preformatted → Background) or the block-level inspector. Gradient layers over the image so both render together.
- **Plugin & theme developers:** No code changes required. The supports are declarative in `block.json` and serialized by `useBlockProps.save()` onto the `<pre>` wrapper. No PHP rendering change.
- **No breaking changes.** The legacy `color.gradients` support remains alongside the new `background.gradient`; existing sites are unaffected.
- **No action required** for existing installations. The new controls simply appear in the Background panel for this block.

## Technical details

The change is confined to `packages/block-library/src/preformatted/block.json`, which adds a new `background` key under `supports`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true
}
```

Because Preformatted is a static block, `useBlockProps.save()` serializes these supports onto the `<pre>` wrapper element at save time; no PHP render callback change is needed. The pre-existing `color.gradients` support is left in place alongside the new `background.gradient` (the two coexist, with the gradient layering over the image per the behavior established in #75859).

Two documentation files are regenerated to reflect the new supports:
- `docs/reference-guides/core-blocks/README.md` — the Preformatted entry's supports list gains `background (backgroundImage, backgroundSize, gradient)`.
- `packages/block-library/src/preformatted/README.md` — a new `background` bullet with the three sub-keys is inserted before the existing `color` entry.

## Contribution

Opened and authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR description notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (3 comments, 0 reactions) with no visible design debate or rejected alternatives; the change follows the established pattern of adding Background-panel supports to a core block.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
