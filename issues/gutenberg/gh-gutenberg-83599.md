# #83599: Term Name: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`bcd397c`](https://github.com/WordPress/gutenberg/commit/bcd397c12654e65484e1e5b0d0c6563b24d0a99e)
- **Discussion:** [#83599](https://github.com/WordPress/gutenberg/pull/83599) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds `dimensions.minWidth` and `dimensions.minHeight` block supports to the Term Name block (`core/term-name`), allowing users to set minimum width and height via Global Styles or the block inspector. This addresses ragged rows in Terms Query grids where term names of varying length cause uneven card heights. The change is part of the broader design-tools consistency effort tracked in #43241.

## Impact

- **Site owners / editors using Terms Query blocks:** Can now set minimum width and height on Term Name via Global Styles → Blocks → Term Name → Dimensions, or per-block in the inspector. No code changes required.
- **Plugin & theme developers:** No action required. The Term Name block is server-rendered and its wrapper is produced by `get_block_wrapper_attributes()`, which serializes the new supports automatically. No render-callback or PHP changes are needed.
- **No breaking changes, deprecations, or removed APIs.** Existing sites render identically until a user explicitly sets a min-width or min-height value.

## Technical details

The functional change is a single addition to `packages/block-library/src/term-name/block.json`:

```json
"dimensions": {
    "minHeight": true,
    "minWidth": true
}
```

Because Term Name is a server-rendered block whose wrapper element is built by `get_block_wrapper_attributes()`, the `--wp--min-height` and `--wp--min-width` custom properties are emitted into the wrapper's `style` attribute automatically when a value is set. No change to the render callback is required.

The render callback always produces content for the wrapper (it returns early only when no term is present, otherwise it prints the term name), so no empty-box guard is needed.

Aspect ratio support was deliberately **not** added; the PR notes that a locked ratio would clip long term names.

Two documentation files are regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — the Term Name entry's supports line now includes `dimensions (minHeight, minWidth)`.
- `packages/block-library/src/term-name/README.md` — a new `dimensions` section with `minHeight: true` and `minWidth: true` is inserted between the `color` and `shadow` entries.

## Contribution

Opened by @aaronrobertshaw and implemented via a Claude Code agent from a predefined task. @shail-mehta resolved merge conflicts before the PR was merged. The record shows 3 comments and 0 reactions with no visible design debate or rejected alternatives beyond the stated decision to omit aspect ratio.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
