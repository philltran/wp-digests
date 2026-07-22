# #79426: Add flex vertical alignment tool to block inspector layout panel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Layout`
- **Merged:** [`3cc40a7`](https://github.com/WordPress/gutenberg/commit/3cc40a77910f9b069d2cdc82a41b0db68cbd11c4)
- **Discussion:** [#79426](https://github.com/WordPress/gutenberg/pull/79426) · 3 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The block editor now exposes vertical alignment controls for flex layout blocks directly in the block inspector’s layout panel. Previously, vertical alignment was only available in the block toolbar. The new control is hidden by default within the `ToolsPanel` to keep the inspector uncluttered, but it syncs bidirectionally with the toolbar control and respects the block’s `layout.verticalAlignment` support.

## Impact

- **Block & theme developers**: No code changes required. Blocks that support flex layout will automatically surface the new inspector control.
- **Site owners & editors**: Vertical alignment is now accessible in the inspector panel alongside other layout settings, improving discoverability for complex layouts.
- **No action required** for existing themes, plugins, or custom blocks.

## Technical details

The change modifies `packages/block-editor/src/layouts/flex.js` to introduce a `ToolsPanelItem` for vertical alignment. It adds an `allowVerticalAlignment` configuration flag to the layout block support schema, alongside helper functions `getDefaultVerticalAlignment()` and `getCompatibleVerticalAlignment()` to handle orientation-aware defaults and invalid state cleanup. The `FlexLayoutVerticalAlignmentControl` component now accepts an `isToolbar` prop: when `false` (inspector), it renders a `ToggleGroupControl` with `ToggleGroupControlOptionIcon` elements mapped from a new `verticalAlignmentOptions` object; when `true` (toolbar), it falls back to `BlockVerticalAlignmentControl`. The control is gated by `allowVerticalAlignment` and hidden by default via `ToolsPanelItem`.

## Contribution

Opened by @tellthemachines and merged with co-authors @culturerich, @ramonjd, @jasmussen, @vcanales, and @jameskoster. The PR notes that vertical alignment was previously omitted from the inspector due to concerns about panel clutter, but the adoption of `ToolsPanel` (which collapses secondary tools by default) resolved that design constraint. The implementation focused on reusing existing alignment logic while adapting the UI component for the inspector context, with no major design debates or rejected alternatives recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
