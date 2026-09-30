# #83583: Login/out: Add minimum width and height support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Login/out`
- **Merged:** [`fcefb0a`](https://github.com/WordPress/gutenberg/commit/fcefb0a0d5cba77fb143be2a111e8ec7fa4bea38)
- **Discussion:** [#83583](https://github.com/WordPress/gutenberg/pull/83583) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Login/out block (`core/loginout`) now supports minimum width and minimum height via the standard `dimensions` block support. The block previously exposed border, background, colour, shadow, spacing, and typography controls but had no minimum-size controls, making it inconsistent with other core blocks. The change is purely additive: a `dimensions` entry is added to the block's `block.json`, and because the block is server-rendered through `get_block_wrapper_attributes()`, no render-callback or PHP changes are required.

## Impact

- **Site owners / editors:** Can now set Minimum width and Minimum height on the Login/out block via Global Styles (Blocks → Login/out → Dimensions) or the block inspector, with independent mobile-state values. No action required unless you want to use the new controls.
- **Theme & plugin developers:** No code changes needed. The block is server-rendered; `get_block_wrapper_attributes()` serialises `min-width` and `min-height` automatically. If you build custom Login/out wrappers or override the block's render, be aware the wrapper element may now carry `min-width`/`min-height` inline styles.
- **Headless / REST consumers:** No schema or route changes. The new styles are emitted as inline CSS on the wrapper element in the rendered HTML.
- **No breaking changes, deprecations, or migrations.**

## Technical details

The diff adds a single `dimensions` object to `packages/block-library/src/loginout/block.json`:

```json
"dimensions": {
    "minHeight": true,
    "minWidth": true
}
```

This is the only functional change. The block's render callback is unchanged; it already calls `get_block_wrapper_attributes()`, which reads the `dimensions` supports and emits `min-width` / `min-height` inline styles on the wrapper `<div>`. Because the render callback always produces content (either the login form or the log-in/log-out link), there is no empty-wrapper edge case to guard against.

Two documentation files are regenerated to reflect the new support:
- `docs/reference-guides/core-blocks/README.md` — the Login/out supports line now lists `dimensions (minHeight, minWidth)`.
- `packages/block-library/src/loginout/README.md` — a new `dimensions` section with `minHeight: true` and `minWidth: true` is inserted between the `color` and `spacing` entries.

No new hooks, filters, REST schema fields, or database changes are introduced.

## Contribution

Authored by @aaronrobertshaw with co-author @ramonjd, and implemented by a Claude Code agent from a predefined task. The PR is part of the broader design-tools consistency effort tracked in #43241. The record carries no human review discussion — the only two comments are automated bot posts (performance metrics and a flaky-test report).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
