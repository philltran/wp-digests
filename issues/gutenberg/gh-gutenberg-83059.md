# #83059: Post Template: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Post Template`
- **Merged:** [`e9354e2`](https://github.com/WordPress/gutenberg/commit/e9354e29e7966d191cfa9aa47cbced4b074633e3)
- **Discussion:** [#83059](https://github.com/WordPress/gutenberg/pull/83059) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Template block (`core/post-template`) now declares the `shadow` block support, allowing users to apply drop shadows via Global Styles or per-instance in the block inspector. This closes a gap where the block already supported color, border, spacing, and typography but lacked shadow control, as part of the broader design-tools consistency effort (related #43241). Because the block is server-rendered through `get_block_wrapper_attributes()`, the support is serialized onto the wrapper element automatically and no PHP change was required.

## Impact

- **Site owners (block themes):** Can now set a shadow on the Post Template block in Global Styles (Blocks → Post Template → Shadow) or override it per instance in the block inspector. No code changes needed.
- **Plugin & theme developers:** No action required. The support is purely declarative in `block.json`; any theme or plugin that already handles the `shadow` support on other blocks will pick this up with no additional work.
- **Headless & REST consumers:** No schema or route changes. The shadow is rendered as inline styles on the server-side wrapper element, so it appears in the rendered HTML but is not a new REST field.
- No breaking changes, deprecations, or migrations.

## Technical details

The functional change is a single addition to `packages/block-library/src/post-template/block.json`:

```json
"supports": {
  "align": ["full", "wide"],
  "anchor": true,
  "color": true,
  "width": true,
  "style": true,
  "shadow": true
}
```

The Post Template block is server-rendered via `get_block_wrapper_attributes()`, which reads the registered supports and serializes the corresponding CSS custom properties (e.g. `--wp--custom--shadow--natural`) and `box-shadow` declarations onto the `<ul>` wrapper element. No PHP template or render-callback change is needed.

Two documentation files were regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — `shadow` added to the supports list for `core/post-template`.
- `packages/block-library/src/post-template/README.md` — a new `shadow: true` entry added under the supports section.

Precedence follows the standard block-supports cascade: a per-instance shadow set in the block inspector overrides the Global Styles shadow; clearing the instance-level shadow reverts to the Global Styles value.

## Contribution

Opened by @aaronrobertshaw and reviewed by @im3dabasia. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. In the discussion, @aaronrobertshaw thanked @im3dabasia for the review and asked that future reviews of design-tool adoption PRs include more detail on what was tested, edge cases considered, and supporting screenshots or video, to build confidence given the potential for edge cases. A self-review checklist in the thread confirms correct behavior for Global Styles application, instance-level override, editor/frontend consistency, and shadow clearing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
