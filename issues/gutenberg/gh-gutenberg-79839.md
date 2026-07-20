# #79839: Editor: allow selecting which block styles to apply globally

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Editor`, `Global Styles`, `[Feature] Design Tools`, `Backport to Gutenberg RC`, `Backported to WP Core`
- **Merged:** [`e790b98`](https://github.com/WordPress/gutenberg/commit/e790b98e7ca3ba0a18892fb72b8e498220209db5)
- **Discussion:** [#79839](https://github.com/WordPress/gutenberg/pull/79839) · 20 comments · 4 reactions
- **Usefulness:** 4/5

## Summary

The block inspector’s **Apply globally** action now opens a review modal instead of immediately pushing all local style changes to Global Styles. The modal lists each modified style with its current and new values, allowing authors to select exactly which styles to propagate to the block type’s global preset. This prevents accidental overwrites and gives designers granular control over which local overrides become site-wide defaults.

## Impact

- **Block & theme developers / site editors:** No code changes required. The editor workflow changes from an all-or-nothing push to a selectable review step.
- **Plugin developers:** No breaking changes to public APIs. Internal editor hooks were refactored, but external consumers are unaffected.
- **Hosting & platform teams:** No configuration or migration needed. The change ships in the editor build.
- **Note:** Deselected styles remain as local block overrides and are not cleared from the block’s attributes. No action required.

## Technical details

- Refactors `packages/editor/src/hooks/push-changes-to-global-styles/index.js` to replace the flat `useChangesToPush` return with grouped `ChangeRow` objects that bundle related paths (e.g., border color, width, style) into a single row.
- Introduces `packages/editor/src/hooks/push-changes-to-global-styles/apply-globally-modal.js`, which renders an `ApplyGloballyModal` using `@wordpress/dataviews`'s `DataViewsPicker` to display `label`, `current`, and `new` columns.
- Adds `packages/editor/src/hooks/push-changes-to-global-styles/format-style-value.js` with utilities (`formatBorderShorthand`, `formatBorderRadius`, `formatSpacingShorthand`, `formatBlockGap`) that collapse per-side/per-corner objects into CSS shorthand strings and resolve preset tokens (e.g., `var:preset|color|vivid-red`) to readable slugs.
- Updates the `pushChanges` flow to accept a filtered subset of rows. Preset attributes (`textColor`, `backgroundColor`, `borderColor`, etc.) are now cleared conditionally only when their corresponding row is selected, preventing accidental wiping of deselected local overrides.
- The modal defaults all rows to selected, disables **Apply** when none are selected, and leaves the block untouched on cancel/Escape.

## Contribution

Opened by @aaronrobertshaw as a proof-of-concept to replace the all-or-nothing "Apply globally" behavior. @annezazu and @jasmussen provided early UX feedback, confirming the current-vs-new value display and row granularity. @jasmussen suggested removing a placeholder chevron icon and noted the modal’s potential as a foundation for broader theme/style switching workflows. The author removed the icon, polished the implementation, and merged the PR targeting WordPress 7.1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
