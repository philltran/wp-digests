# #83184: Tab Panels: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`495afd3`](https://github.com/WordPress/gutenberg/commit/495afd36268375439b724985e47ee1cf777e45f0)
- **Discussion:** [#83184](https://github.com/WordPress/gutenberg/pull/83184) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panels block (`core/tab-panels`) now declares the `shadow` block support, allowing users to apply box-shadow presets via Global Styles or the block inspector. This closes a gap where the block already supported color, border, spacing, and typography but had no shadow control, as part of the broader design-tools consistency effort tracked in #43241. Because Tab Panels is a static block, the support is serialized onto the wrapper element by `useBlockProps.save()` and requires no PHP render-callback change.

## Impact

- **Site owners / editors:** Tab Panels can now receive a shadow preset (Natural, Deep, Sharp, Crisp, etc.) from Global Styles or the block-level Shadow control, with full responsive (mobile/tablet/desktop) support. No code changes needed.
- **Theme & plugin developers:** No action required. The new support is additive; existing themes and plugins that style `core/tab-panels` are unaffected. If a theme previously applied its own `box-shadow` to the Tab Panels wrapper, the new Global Styles / block-level shadow will layer on top via the standard cascade.
- **Headless / REST consumers:** No schema or route changes. The serialized block markup gains a `style="box-shadow: …"` attribute on the wrapper when a shadow is set, same as any other shadow-supported block.

## Technical details

The functional change is a single line in `packages/block-library/src/tab-panels/block.json`, adding `"shadow": true` to the `supports` object:

```json
// before
"supports": {
  "color": { "background": true, "text": true, "heading": true, "link": true },
  "spacing": { "padding": true },
  "typography": { "fontSize": true }
}

// after
"supports": {
  "color": { "background": true, "text": true, "heading": true, "link": true },
  "shadow": true,
  "spacing": { "padding": true },
  "typography": { "fontSize": true }
}
```

Because `core/tab-panels` is a static block (no `render` callback in PHP), the shadow value is written into the wrapper element's inline `style` attribute by `useBlockProps.save()` at save time and read back by `useBlockProps()` in the editor. No changes to `render.php`, no new hooks, no DB or REST schema changes.

Two documentation files are regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — the Tab Panels entry's **Supports** line gains `shadow`.
- `packages/block-library/src/tab-panels/README.md` — a new bullet for `shadow: true` is inserted under the `supports` list.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The two comments on the PR are both from the `github-actions[bot]` (performance metrics and flaky-test report); no human review discussion or design debate is recorded. Merged at `495afd3`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
