# #78766: Block Editor: Refactor Inserter to a function component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Code Quality`, `[Package] Block editor`
- **Merged:** [`47326be`](https://github.com/WordPress/gutenberg/commit/47326be7d961b3dfd14202d816385afec4ae405e)
- **Discussion:** [#78766](https://github.com/WordPress/gutenberg/pull/78766) · 6 comments · 0 reactions

## Summary

The `Inserter` component in `@wordpress/block-editor` has been converted from a class component to a function component. As part of the refactor, the internal `defaultRenderToggle` factory function is promoted to a named `InserterToggle` component; when the `renderToggle` prop is not provided, `<InserterToggle>` now renders directly. Unreachable guard clauses were also removed as opportunistic cleanup. The PR carries no public API or behavior changes and yields a 223 B reduction in `build/scripts/block-editor/index.min.js`.

## Impact

**Plugin & theme developers**
- No action required. The `renderToggle` prop on `<Inserter>` is preserved and behaves identically.
- No deprecations, no removed APIs, no changed prop signatures.

**Hosting & platform teams bundling Gutenberg**
- `build/scripts/block-editor/index.min.js` shrinks by 223 B (−0.06%). No cache-busting concern beyond the normal Gutenberg release cadence.

## Technical details

The core change is a class-to-function-component conversion of `Inserter` inside `@wordpress/block-editor`.

The previously internal `defaultRenderToggle` factory function is extracted and promoted to a proper named component, `InserterToggle`. The toggle-rendering branch is simplified: when no `renderToggle` prop is supplied, the component renders `<InserterToggle>` directly rather than invoking the factory. The `renderToggle` prop codepath is unchanged.

Unreachable guard conditions identified during the refactor were removed, contributing to the small bundle size reduction.

No new hooks, filters, REST schema fields, or `block.json` keys are introduced or modified.

## Contribution

Opened and merged by @Mamaduka (commit `47326be`). The PR drew six comments, all from the automated size-check bot; no design debate or alternative approaches are documented in the discussion. The author noted the work was assisted by Claude.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
