# #83177: Pullquote: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Pullquote`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`8272bc5`](https://github.com/WordPress/gutenberg/commit/8272bc5735ab9edb078a5df85aaa34d97e321e73)
- **Discussion:** [#83177](https://github.com/WordPress/gutenberg/pull/83177) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Pullquote block (`core/pullquote`) now supports the `shadow` block support, enabling box-shadow styling via Global Styles and the block inspector. This closes a gap where the block already supported background, color, border, dimensions, spacing, and typography but lacked shadow control. The change is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Theme & site developers:** The Pullquote block's wrapper `<figure>` element will now receive shadow CSS from Global Styles and per-block overrides, just like other blocks that already support shadow. No code changes required; existing themes that do not set a shadow will render identically.
- **Block developers:** No new API or hook is introduced. The `shadow` key in `block.json` is the standard mechanism already used by other core blocks. No PHP render-callback change is needed because Pullquote is a static block and `useBlockProps.save()` serializes the support onto the wrapper element.
- **No action required** for existing sites or plugins. The change is purely additive.

## Technical details

The functional change is a single line in `packages/block-library/src/pullquote/block.json`, adding `"shadow": true` to the `supports` object:

```json
"supports": {
  "dimensions": { "minHeight": true },
  "shadow": true,
  "spacing": { "margin": true, "padding": true }
}
```

Because Pullquote is a static block (no dynamic render callback), the shadow support is serialized directly onto the wrapper `<figure>` element by `useBlockProps.save()` at save time. No PHP-side change is required.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the Pullquote supports list.
- `packages/block-library/src/pullquote/README.md` — a new `shadow: true` entry added under the supports section.

The support participates in the standard cascade: Global Styles (including responsive/mobile states) set a base shadow, and a per-block shadow chosen in the block inspector overrides it at the instance level.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @andrewserong. The PR notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives recorded. It was merged as commit `8272bc5`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
