# #80080: Responsive editing: Add a Tooltip to the viewport / states badge

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`6ce7a2c`](https://github.com/WordPress/gutenberg/commit/6ce7a2c68b93103e1b07f18cddd96f26baf7ca9e)
- **Discussion:** [#80080](https://github.com/WordPress/gutenberg/pull/80080) · 18 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor now renders a tooltip on the viewport and pseudo-state badges in the Global Styles sidebar when responsive editing or style states are active. This enhancement clarifies that applied styles are scoped to the selected viewport or state, reducing editor confusion. An initial proposal to add a close button for exiting responsive editing was removed during review due to ambiguous UX expectations, leaving only the tooltip implementation.

## Impact

- **Block & theme developers:** No code changes or migration steps required. The change is purely UI-level within the block editor.
- **Site owners & editors:** Improved clarity when applying responsive or state-specific styles in the Global Styles sidebar, making it explicit which viewport or pseudo-state is currently active.
- **No breaking changes or deprecations.** Existing badge rendering remains compatible; the component now conditionally wraps badges in a `Tooltip` component.

## Technical details

The change modifies `packages/block-editor/src/components/global-styles/state-control-badges.js` to inject a `tooltipText` property into the `activeStates` array for both viewport and pseudo-state selections. Each badge is now conditionally wrapped in `Tooltip.Root`, `Tooltip.Trigger`, and `Tooltip.Popup` from `@wordpress/ui`. A `VisuallyHidden` span is rendered inside the badge to expose the tooltip text to screen readers. The component also introduces the `.block-editor-global-styles-state-control__badge-tooltip-trigger` class in `style.scss` to ensure proper `inline-flex` layout. Unit tests were added to verify that both the visible badge and the tooltip contain the expected localized text (e.g., `Style changes apply to the Tablet viewport.`).

## Contribution

Opened by @andrewserong as a proof-of-concept to improve state legibility in the Global Styles sidebar. Initial iterations included a close button to exit responsive editing, but reviewers (@tellthemachines, @annezazu, @ramonjd) highlighted UX ambiguity around whether the button should toggle the viewport, disable responsive editing, or both. The scope was narrowed to a tooltip-only implementation, which was merged with co-authors @ramonjd, @t-hamano, @talldan, @tellthemachines, and @annezazu.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
