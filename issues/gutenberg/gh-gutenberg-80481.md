# #80481: Block Editor: Reflect inherited Global Styles values in block inspector controls

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`3dced39`](https://github.com/WordPress/gutenberg/commit/3dced3957c456cfba29f2d5f35a7afe4d5b5d310)
- **Discussion:** [#80481](https://github.com/WordPress/gutenberg/pull/80481) · 5 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

This pull request backports a block editor enhancement that makes inspector controls in standard block-supports panels (Typography, Dimensions, Border, Color, Background, Filters) display inherited Global Styles values when no local override is set. Instead of showing empty or default states, controls now render the inherited value as a placeholder or preselected option, and mark the label with a dotted underline. When a user applies a local override, a reset affordance appears to revert the control back to the inherited value, improving visual feedback and reducing guesswork when working with theme or site-wide style inheritance.

## Impact

- **Block & theme developers:** No breaking changes. The enhancement modifies internal block editor UI components (`DimensionControl`, `AspectRatioTool`, `BackgroundImagePanel`) to support inheritance indicators. Existing custom controls using these components will automatically gain the new behavior if they pass the updated props, but no migration is required.
- **Site owners & editors:** Improved clarity in the block inspector when Global Styles are applied. Controls will visually indicate inherited values and provide explicit reset buttons to clear local overrides.
- **Platform/Hosting:** No action required. This is a client-side UI enhancement with no database, REST API, or PHP changes.

## Technical details

The diff backports the original trunk implementation to `wp/7.1`, resolving a merge conflict in `packages/block-editor/src/components/global-styles/typography-panel.js`. Key behavioral changes include:
- Replacing `ToolsPanelItem` with `InheritanceToolsPanelItem` (imported from `../global-styles/inheritance`) in `aspect-ratio-tool.js` and `background-panel.js` to render inheritance indicators.
- Introducing `hasLocalOverride`, `isInherited`, and `showInheritanceLabelIndicators` props to control UI rendering. When `hasLocalOverride` is true, `background-image-control/index.js` conditionally renders `InheritanceResetButton` instead of the plain reset button.
- Updating `resetBackground` in `background-image-control/index.js` to preserve gradient settings when clearing a background image: `setImmutably( value, [ 'background' ], { gradient: value?.background?.gradient } )`.
- Adding `placeholder` and `className` props to `dimension-control/index.js` so inherited values can be displayed as placeholders.
- Switching `switch-section-style.js` to use `getVariationStyle` from `globalStylesEnginePrivateApis` via `unlock()` instead of the legacy `getVariationStylesWithRefValues` hook.

## Contribution

Opened to resolve a merge conflict in `typography-panel.js` caused by a concurrent PR not yet landing in the release branch. Reviewers validated the conflict resolution and bundle size impact before merging. The backport applies the exact trunk diff without modification, and no alternative implementation strategies were debated during this cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
