# #82840: Block Inspector: Stabilize display of inherited global styles and keep UI changes behind experiment

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`e8ff6b6`](https://github.com/WordPress/gutenberg/commit/e8ff6b6c7a89968e9a2c54d6148f91b1b5eb738f)
- **Discussion:** [#82840](https://github.com/WordPress/gutenberg/pull/82840) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Block inspector controls in the standard block-supports panels (Typography, Dimensions, Border, Color, Background, Filters) now show the value a block inherits from Global Styles when nothing is set on the block, without opting in to an experiment. Only the visual indicators stay behind the `gutenberg-global-styles-inheritance-ui` experiment: the dotted underline on an inherited label and the dot that resets a local override. The PR is mostly a deletion of an early return in `useResolvedStyle` that had disabled the cascade resolution when the experiment was off.

## Impact

- **Site owners / editors**
  - Controls such as Line height now display the theme's inherited value (e.g. 1.2 for headings in Twenty Twenty-Four) instead of appearing empty. Resetting a value returns to the theme value, not to empty.
  - A block inheriting a background image from Global Styles now shows the size, position and repeat controls.
  - A block with a text colour and no link colour set locally or inherited now starts with the link colour tracking the text colour. Previously it stayed unset.
- **Plugin & theme developers**
  - `isGlobalStylesInheritanceEnabled` is renamed `isGlobalStylesInheritanceIndicatorUIEnabled` in `components/global-styles/inheritance`. It is not shown to be a public export, but anything importing it internally must be updated.
  - Tests or code that assumed the inspector shows only locally-set values when the experiment is off will see different behavior.
  - Saved markup should carry only what the user sets (a testing step in the PR checks this).
- **Experiment users**
  - The experiment id is unchanged, so anyone who opted in stays opted in. The label and description now cover the indicators only.
- **Headless / REST / hosting:** No impact.

## Technical details

Changes are in `packages/block-editor/src/components/global-styles/` and related files:

- `inherited-value-context.js`: the `useResolvedStyle` early return, which yielded `NO_RESOLVED_STYLE` (`{ value: undefined, sources: undefined }`) when the experiment was off, is removed along with the constant and the import. With it gone, the cascade is always resolved and panels receive real `inheritedValue`s instead of falling back to `inheritedValue = value`.
- `inheritance/index.jsx`: `isGlobalStylesInheritanceEnabled` is renamed `isGlobalStylesInheritanceIndicatorUIEnabled`. It still returns `!! window.__experimentalGlobalStylesInheritanceUI` and now answers only whether indicators render.
- Panels (`background-panel.jsx`, `border-panel.jsx`, `color-gradient-dropdown-item.jsx`, `color-panel.jsx`, `dimensions-panel.jsx`, `filters-panel.jsx`, `background-image-control/index.jsx`): the `showInheritanceLabelIndicators` prop default now calls the renamed function. The prop gate is unchanged.
- `lib/experimental/experiments/load.php`: label becomes "Global Styles inheritance indicators in the block inspector" and the description is reworded. The id `gutenberg-global-styles-inheritance-ui` is kept because it is the stored option key.
- Per the PR description, `setTextColor` now always decides link colour sync from `shouldSyncLinkColor`, dropping the older comparison path that only held while `inheritedValue` fell back to `value` (that hunk is in the truncated part of the diff).
- `hasImageValue` in `background-image-control/index.jsx` combines local and inherited images and is not gated, hence the background size/position/repeat controls appearing for inherited images.
- Tests: `inherited-value-context-core.jsdom.test.js` now asserts the root, block and element layers resolve with the experiment off. `typography-panel-core.browser.test.jsx` gains a case checking the inherited value shows on the control with no inheritance classes, and adds a third palette preset (green) to distinguish "left alone" from "synced" link colour.
- `packages/block-editor/CHANGELOG.md` gets two entries.

## Contribution

Follow-up to #82520, which asked for the inherited value to be shown ungated early in the 7.2 cycle so feedback could settle the still-unresolved local override indicator. Review discussion with @ramonjd centered on two edge cases: adjusting a partly inherited control (background size/repeat/position, and spacing with unlinked sides) writes sibling inherited values into the block. @aaronrobertshaw framed both as one policy question, argued that adjusting a control means overriding the theme for that group, and noted spacing is stored as `var:preset` strings so preset changes still propagate. @ramonjd's remaining concern was hardcoding an inherited value on the block, but he accepted the approach, and a separate issue was offered to document the outcome. The PR notes it was written with Claude Code against a reviewed spec.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
