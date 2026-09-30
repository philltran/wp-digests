# #83202: Breadcrumbs: Add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Breadcrumbs`
- **Merged:** [`6c8b05b`](https://github.com/WordPress/gutenberg/commit/6c8b05b17bd092539d4889e904483f14c1ea037b)
- **Discussion:** [#83202](https://github.com/WordPress/gutenberg/pull/83202) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Breadcrumbs block (`core/breadcrumbs`) now supports the Background panel's gradient controls via the `background.gradient` block support. Previously the block only had the legacy `color.gradients` support, meaning the modern Background panel gradient picker was unavailable. This brings Breadcrumbs in line with other core blocks as part of the ongoing design-tools consistency work (related to #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the new support serializes onto the wrapper automatically with no PHP change.

## Impact

- **Theme & site builders:** The Breadcrumbs block now exposes the Background → Gradient control in both Global Styles and the block inspector. No code changes required; the control appears automatically once the block support is registered.
- **Plugin & theme developers:** No breaking change. The legacy `color.gradients` support remains alongside the new `background.gradient` support, so existing themes that style breadcrumbs via the old gradient mechanism are unaffected.
- **No action required** for existing sites. The change is purely additive; blocks without a gradient set render exactly as before (the wrapper returns an empty string when there are no breadcrumb items, so no empty box is painted).

## Technical details

The functional change is a single addition to `packages/block-library/src/breadcrumbs/block.json`:

```json
"background": {
    "gradient": true
}
```

This is inserted into the existing `supports` object, alongside the pre-existing `color.gradients: true` entry. Because `core/breadcrumbs` is server-rendered via `get_block_wrapper_attributes()`, the block-supports serialization pipeline picks up the new key and emits the appropriate CSS custom properties on the wrapper element — no change to `render.php` or any PHP template is needed.

Two documentation files are regenerated to reflect the new support:

- `docs/reference-guides/core-blocks/README.md` — the Breadcrumbs supports line gains `background (gradient)`.
- `packages/block-library/src/breadcrumbs/README.md` — a new `background` → `gradient: true` entry is added under the supports list.

The legacy `color.gradients` support is intentionally retained so that themes relying on the older gradient mechanism continue to work.

## Contribution

Authored by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion thread contains only the two automated bot comments (performance metrics and contributor attribution); no design debate or alternative approaches are recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
