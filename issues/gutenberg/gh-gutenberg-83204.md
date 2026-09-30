# #83204: Code: Add background image, size, and gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Code`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`e6130cc`](https://github.com/WordPress/gutenberg/commit/e6130cc9b2b0b841177aa3347be5f09d23d6013e)
- **Discussion:** [#83204](https://github.com/WordPress/gutenberg/pull/83204) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Code block (`core/code`) now supports background image, background size, and gradient via the `background` supports key in its `block.json`. This closes a gap where the block already supported color, border, shadow, spacing, and typography but lacked the Background panel's image, size, and gradient controls. The change is part of the broader design-tools consistency effort (related to #43241) to bring core blocks in line with one another. Because Code is a static block, the new supports are serialized onto the `<pre>` wrapper by `useBlockProps.save()` and require no PHP-side change.

## Impact

- **Site owners / content editors:** Can now set a background image, its size (e.g. Contain, Cover), and a gradient on Code blocks through Global Styles (Blocks → Code → Background) or the block inspector. Gradient layers over the image so both render together.
- **Plugin & theme developers:** No action required. The change is purely additive to `block.json` supports. No hooks, REST routes, or PHP render functions changed.
- **Headless & REST consumers:** No schema or serialization change beyond the existing `useBlockProps.save()` output on the `<pre>` element. No new attributes are exposed in the block markup beyond what the supports system already emits.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The diff modifies `packages/block-library/src/code/block.json`, adding a `background` object under `supports`:

```json
"background": {
  "backgroundImage": true,
  "backgroundSize": true,
  "gradient": true,
  "__experimentalDefaultControls": {
    "backgroundImage": true,
    "gradient": true
  }
}
```

The `__experimentalDefaultControls` sub-key enables the image and gradient controls in the block inspector by default (background size is available but not shown by default). The pre-existing `color` supports entry (which includes `gradients` for the legacy gradient picker) is untouched, so both `color.gradients` and `background.gradient` coexist — the same pattern used by the Quote and Verse blocks.

Because `core/code` is a static block, no `render.php` or `render_callback` change is needed; `useBlockProps.save()` serializes the new CSS custom properties onto the `<pre>` wrapper. The gradient is layered over the background image per the behavior established in #75859.

Two documentation files are regenerated to reflect the new supports: `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/code/README.md`.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The record shows only 2 comments (both from the GitHub Actions bot posting performance metrics and a flaky-test report) and no design debate or alternative approaches discussed before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
