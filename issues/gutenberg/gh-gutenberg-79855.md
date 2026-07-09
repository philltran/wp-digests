# #79855: Block Editor: Share block-bindings context assembly between call sites

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @cbravobernal
- **Labels:** `[Package] Block editor`, `[Type] Feature`, `[Feature] Block bindings`
- **Merged:** [`f850ad2`](https://github.com/WordPress/gutenberg/commit/f850ad24465cb65b2eb602c2a85297f2fedcb6e4)
- **Discussion:** [#79855](https://github.com/WordPress/gutenberg/pull/79855) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Block Editor refactors context assembly for block bindings sources into a shared `getBlockBindingsContext` helper in `packages/block-editor/src/utils/block-bindings.ts`. This replaces an inline pattern in `block-edit/edit.js` that inadvertently assigned `undefined` values to declared-but-unprovided keys and risked mutating a shared module-level constant. The refactor ensures only present context entries are copied, returns a stable empty object when nothing matches, and provides a reusable foundation for upcoming innerBlocks binding resolvers.

## Impact

- **Block & plugin developers:** No action required. Public `usesContext` behavior remains aligned, but undeclared keys now safely resolve as absent rather than existing with `undefined` values.
- **Hosting & platform teams:** No changes needed. Internal editor rendering pipeline change only; no migration or configuration updates required.

## Technical details

The unified diff moves context assembly logic from `packages/block-editor/src/components/block-edit/edit.js` into a new export in `packages/block-editor/src/utils/block-bindings.ts`. The `getBlockBindingsContext( blockContext, blockTypeUsesContext, sources )` function explicitly checks `key in blockContext` before assigning values, preventing the previous `Object.fromEntries()` pattern from creating own `undefined`-valued keys. It returns a module-level `EMPTY_CONTEXT` reference when no matching entries exist, ensuring reference-stability for consumers that compare objects by identity. Unit tests added to the newly created `packages/block-editor/src/utils/test/block-bindings.js` cover the new helper and previously untested `hasPatternOverridesDefaultBinding` and `replacePatternOverridesDefaultBinding` utilities.

## Contribution

Merged as `f850ad2` by @cbravobernal with co-authorship from @sirreal. Extracted from #79852 to prevent duplicate context assembly logic before implementing innerBlocks binding resolvers. The change was reviewed as a straightforward correctness refactor, with the author noting that sharing the helper keeps attribute bindings and future innerBlocks bindings aligned instead of requiring inline duplication.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
