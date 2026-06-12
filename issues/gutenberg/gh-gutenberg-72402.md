# #72402: `ColorPalette`: don't render when custom colors disabled and no colors passed

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @n8finch
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Package] Block editor`, `First-time Contributor`
- **Merged:** [`c2cf65a`](https://github.com/WordPress/gutenberg/commit/c2cf65abe8e801daf84fea59cbc991bf8bcc0c5f)
- **Discussion:** [#72402](https://github.com/WordPress/gutenberg/pull/72402) · 10 comments · 1 reactions

## Summary

The `ColorPalette` component now returns early when custom colors are disabled, no color presets exist, and no actions (like clear buttons) are configured. This prevents an empty UI wrapper from rendering in the Block Editor, eliminating unnecessary DOM nodes and visual artifacts when theme.json explicitly opts out of custom color selection.

## Impact

- **Theme & plugin developers:** No API changes or breaking modifications. Existing `theme.json` configurations remain fully compatible.
- **Block editor & UI consumers:** The `ColorPalette` component will now skip rendering entirely under the specified conditions, reducing layout shifts and DOM bloat when color pickers are intentionally disabled.
- **No action required** for standard deployments; existing integrations relying on `disableCustomColors` or preset-defined palettes will continue to function as expected.

## Technical details

- **Core logic update:** Added a guard clause directly inside `packages/components/src/color-palette/index.tsx` (UnforwardedColorPalette):
  ```tsx
  if ( disableCustomColors && colors.length === 0 && ! actions ) {
    return null;
  }
  ```
- **Prop passing alignment:** Updated `packages/block-editor/src/components/global-styles/border-panel.js` to explicitly pass `disableCustomColors={ !areCustomSolidsEnabled }` to `BorderBoxControl`, closing the gap where `ColorPanelDropdown` already respected this setting.
- **Test coverage:** Added assertions in `packages/components/src/color-palette/test/index.tsx` verifying that the component renders nothing when all guard conditions are met, while preserving the clear button's visibility when applicable.
- The diff consolidates early-exit logic into `ColorPalette` itself rather than duplicating theme.json checks upstream, aligning with reviewer feedback to avoid redundant conditionals.

## Contribution

Opened by @n8finch to resolve issue #59567, initially proposing a fix to pass the missing `disableCustomColors` prop through `BorderBoxControl`. During peer review, @ciampo and @aaronrobertshaw identified that replicating theme.json checks upstream duplicated logic already handled by `ColorPalette`. The implementation was refactored to move the guard directly into the component, returning `null` when conditions aligned. After addressing linting requirements and adjusting test assertions to validate the early return without duplicating existing checks, the PR was merged with several co-authors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
