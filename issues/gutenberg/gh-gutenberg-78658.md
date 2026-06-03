# #78658: Hide block variation selector when style state is selected

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Block Variations`
- **Merged:** [`5aa7478`](https://github.com/WordPress/gutenberg/commit/5aa7478e229cb92179d9ad420566161cf1bc4e62)
- **Discussion:** [#78658](https://github.com/WordPress/gutenberg/pull/78658) · 2 comments · 0 reactions

## Summary

The block variation switcher in the Inspector sidebar is now hidden whenever a responsive style state (tablet or mobile) is active on a block. Previously, selecting a non-default viewport state via the state dropdown near the block title left the variation switcher visible, implying that per-breakpoint variation switching was possible — it is not. The fix suppresses the `BlockVariationTransforms` component while any block style state is selected, eliminating the misleading UI.

## Impact

**Plugin & theme developers**
- No action required. This is a pure editor UX fix with no API surface change.
- Developers who render `BlockVariationTransforms` independently in custom inspector panels are unaffected — the component itself is unchanged.

**Site owners / content editors**
- The variation picker (e.g. the layout-style switcher on Heading or Group blocks) disappears while tablet or mobile state is active in the block inspector. It reappears upon returning to the default state. No variation data or saved content is altered.

**Hosting & platform / headless**
- No action required. Frontend output and stored block markup are unaffected.

## Technical details

The change is a single conditional in `packages/block-editor/src/components/block-inspector/index.js`, inside the `BlockInspectorSingleBlock` component.

**Before:**
```jsx
<BlockVariationTransforms blockClientId={ renderedBlockClientId } />
```

**After:**
```jsx
{ ! isBlockStyleStateSelected && (
    <BlockVariationTransforms
        blockClientId={ renderedBlockClientId }
    />
) }
```

The `isBlockStyleStateSelected` boolean is already computed earlier in `BlockInspectorSingleBlock` and drives the adjacent `StyleStateInspectorSlots` conditional. This change reuses that same flag to gate `BlockVariationTransforms`. No new state, hooks, or props are introduced. The `BlockVariationTransforms` component itself (`@wordpress/block-editor`) is unmodified; the build size reports zero net change.

## Contribution

Opened and authored by **@talldan** on 2026-05-26. Merged at commit `5aa7478`. Review attribution includes **@Mamaduka**. The PR description notes use of OpenCode/Codex during development. Discussion was minimal — two bot comments, no substantive design debate recorded in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
