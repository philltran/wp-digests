# #80495: Global Styles: Resolve per-level heading element styles in block inspector controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Global Styles`, `[Feature] Design Tools`, `Backported to WP Core`
- **Merged:** [`76749bf`](https://github.com/WordPress/gutenberg/commit/76749bf12fb3beb854220c78f51f160c1df437d7)
- **Discussion:** [#80495](https://github.com/WordPress/gutenberg/pull/80495) · 11 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

The block editor’s Global Styles inheritance logic now correctly resolves per-level heading element styles (e.g., `h2`–`h6`) in the inspector sidebar. Previously, only the generic `heading` element layer was applied, causing blocks like `core/heading` and title blocks to ignore level-specific typography and color rules set in Global Styles. This fix ensures the inspector reflects the exact inherited values before a local override is applied, aligning the editor experience with front-end rendering.

## Impact

- **Theme & plugin developers:** No action required. If you define level-specific heading styles in Global Styles (`styles.elements.h2`–`h6`), the inspector will now accurately display those inherited values without code changes.
- **Block developers:** Blocks that use the `level` attribute or inherit it via block context (e.g., `core/accordion-heading`) will now correctly resolve element layers in the inspector.
- **Site owners:** Heading blocks in the editor will show inherited styles from Global Styles > Headings > specific levels, matching the front-end output.
- **No migration or configuration changes needed.**

## Technical details

The change modifies two core files to thread heading-level resolution into the global styles engine:

- `packages/block-editor/src/components/global-styles/inherited-value-context.js`: Replaced `useOwnVariation` with `useVariationAndElements`, which reads the `level` attribute via `getBlockAttributes` (scoped to `clientId` from `useBlockEditContext`) or block context for `core/accordion-heading`. It passes an ordered `elements` array (e.g., `['heading', 'h2']`) to `resolveStyle()`.
- `packages/global-styles-engine/src/resolve-style.ts`: Updated `ResolveStyleContext` to accept `elements?: string[] | null`. Removed the hardcoded `BLOCK_TO_ROOT_ELEMENT` map. `computeResolvedStyle` now iterates over the `elements` array, folding each root-level `styles.elements.*` layer into the cascade in low-to-high precedence order.

Level `0` (paragraph rendering) correctly bypasses heading element layers. `core/accordion-heading` falls back to level `3` when context is absent. Tests verify context inheritance, level `0` handling, and precedence ordering.

## Contribution

Opened as a targeted follow-up to the initial global styles inheritance PR, this change was authored by @andrewserong and refined through review by @ramonjd and @getdave. The scope was deliberately kept narrow to heading-level resolution, with ancestor-style resolution and `link` element support explicitly deferred to future iterations. The PR was merged and backported to the `wp/7.1` branch for inclusion in the next core release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
