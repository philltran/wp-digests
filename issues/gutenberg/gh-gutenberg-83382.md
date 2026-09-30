# #83382: No Results: Add heading and button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] No Results`
- **Merged:** [`d2b13a5`](https://github.com/WordPress/gutenberg/commit/d2b13a535310a973cffa07ce5f23c71e52645ef7)
- **Discussion:** [#83382](https://github.com/WordPress/gutenberg/pull/83382) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The No Results block (`core/query-no-results`) now supports `color.heading` and `color.button` in its `block.json` supports, adding heading and button colour controls to the block inspector and Global Styles. This brings the block in line with other container blocks (e.g. Columns) as part of the design tools consistency effort. No PHP change is required because the block is dynamic and already uses `get_block_wrapper_attributes()`; the element colour CSS was already being serialized on the front end — this PR only exposes the editor controls.

## Impact

- **Theme & block developers:** No code changes needed. If you were previously working around the missing controls by applying heading/button colours via Global Styles element defaults or custom CSS, those overrides will now be superseded by per-block values set in the inspector.
- **Site editors / content authors:** New colour pickers appear under Block Inspector → Styles → Elements and under Global Styles → Blocks → No Results → Elements for heading and button colours, including mobile responsive states.
- **No action required** for existing sites. The change is additive; previously unset values fall back to theme defaults.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/query-no-results/block.json`** — the core change. Two keys are added to the existing `color` supports object:

```json
"color": {
    "gradients": true,
    "link": true,
    "heading": true,
    "button": true
}
```

2. **`packages/block-library/src/query-no-results/README.md`** — regenerated docs listing the new `heading` and `button` sub-keys under `color`.

3. **`docs/reference-guides/core-blocks/README.md`** — the auto-generated core-blocks reference is updated to reflect the new supports list.

No changes to `render.php`, `index.js`, or any PHP file. The PR description notes that `lib/block-supports/elements.php` already serializes element colours for any block that does not explicitly opt out, so the front-end CSS output was already correct; the `block.json` supports are what gate the editor UI (inspector controls and Global Styles element pickers). The pattern mirrors the earlier Columns block change (#54104).

## Contribution

Opened by @aaronrobertshaw with co-authorship from @andrewserong. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The record shows no substantive design debate or alternative approaches — the change is a direct mirror of an established pattern (Columns, #54104) and was merged without visible pushback.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
