# #79192: Icon block: Move flip controls to toolbar group.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Icon`
- **Merged:** [`240812b`](https://github.com/WordPress/gutenberg/commit/240812bd94367178367c002510993779ae20b293)
- **Discussion:** [#79192](https://github.com/WordPress/gutenberg/pull/79192) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Icon block's transformation controls (flip horizontal, flip vertical, and rotate) have been moved from the `other` toolbar group to the `block` toolbar group, aligning its edit UI with the Image block. This reorders the block controls so transformation buttons appear first, followed by the Replace/Choose icon action, while keeping the content-only mode label editor in a separate group. The change improves cross-block consistency without modifying block attributes, save behavior, or public APIs.

## Impact

- **Block & theme developers:** No breaking changes to block attributes, `block.json`, or save logic. The `BlockControls` rendering order changes internally, but this does not affect block registration or theme compatibility.
- **Plugin developers:** No impact unless you are programmatically injecting or overriding `BlockControls` for the Icon block via filters/hooks (uncommon).
- **Site owners & editors:** Toolbar layout shifts slightly: flip/rotate buttons now appear before the Replace/Choose icon button. No migration or configuration needed.
- **No action required** for standard upgrades or existing sites.

## Technical details

The change modifies `packages/block-library/src/icon/edit.js`. It removes the `ToolbarGroup` import and restructures the `BlockControls` JSX to separate controls by group:
- Transformation buttons (`flipHorizontal`, `flipVertical`, `rotation`) are now rendered directly inside a `<BlockControls group="block">`.
- The Replace/Choose icon button moves to a `<BlockControls group="other">`.
- The content-only mode `ariaLabel` `DropdownMenu` remains in a separate `<BlockControls group="other">` but is rendered after the Replace button to match the desired control flow.

Before/after pattern (simplified):
```jsx
// Before
<BlockControls group="other">
  <ToolbarGroup>
    <ToolbarButton onClick={setInserterOpen}>Replace/Choose</ToolbarButton>
  </ToolbarGroup>
</BlockControls>
{ icon && <BlockControls group="other"><ToolbarGroup>...flip/rotate...</ToolbarGroup></BlockControls> }

// After
{ icon && <BlockControls group="block">
  <ToolbarButton>Flip H</ToolbarButton>
  <ToolbarButton>Flip V</ToolbarButton>
  <ToolbarButton>Rotate</ToolbarButton>
</BlockControls> }
<BlockControls group="other">
  <ToolbarButton onClick={setInserterOpen}>Replace/Choose</ToolbarButton>
  { isContentOnlyMode && icon && <DropdownMenu>...</DropdownMenu> }
</BlockControls>
```
No new hooks, filters, REST schema fields, or database changes are introduced. The `ariaLabel` attribute and `__experimentalToolsPanel` integration remain unchanged.

## Contribution

Opened as a follow-up to standardize the Icon block's toolbar with the Image block. During review, a reviewer noted that the Replace button should render after the `block` group controls to match the visual flow and questioned the conditional `group` prop logic. A follow-up commit was pushed to reorder the groups and simplify the conditional rendering, which was then merged after checks cleared.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
