# #82751: Cover Block: Add a Media section to the block inspector panel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Package] Block library`
- **Merged:** [`62008a4`](https://github.com/WordPress/gutenberg/commit/62008a4dad56eb2dcd4aac8b1d84d6d12b88e098)
- **Discussion:** [#82751](https://github.com/WordPress/gutenberg/pull/82751) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Cover block's editor inspector now always displays a **Media** section (a `ToolsPanel` containing a `MediaControl`) when the block is selected, regardless of whether media has been set. Previously, media could only be added from the toolbar, and the sidebar showed nothing for an empty Cover. This brings the Cover in line with the Image, Site Logo, and Media & Text blocks, which already expose a sidebar media control. The media selection path is identical to the toolbar path, so the resulting block attributes are the same either way.

## Impact

- **Site owners / editors:** An empty Cover block now shows an "Add media" control in the sidebar's Settings tab, alongside the existing toolbar button. No configuration or migration required.
- **Plugin & theme developers:** No new public API, hook, or attribute is introduced. The `MediaControl` component from `packages/block-library/src/utils/media-control` is reused internally. If you have custom tests or snapshots of the Cover inspector, the new `ToolsPanel` with label "Media" will appear whenever the block is selected.
- **No action required** for most developers. The change is additive UI within the block's own edit component.

## Technical details

The core change is in `packages/block-library/src/cover/edit/inspector-controls.jsx`. A new `mediaInspectorPanel` variable is computed and rendered before the existing conditional settings panel:

```jsx
const mediaInspectorPanel = isSelected ? (
  <InspectorControls group="content">
    <ToolsPanel
      label={ __( 'Media' ) }
      resetAll={ onClearMedia }
      dropdownMenuProps={ dropdownMenuProps }
    >
      <ToolsPanelItem
        label={ __( 'Media' ) }
        hasValue={ () => !! url || !! useFeaturedImage }
        onDeselect={ onClearMedia }
        isShownByDefault
      >
        <MediaControl
          mediaId={ id }
          mediaUrl={ url }
          filename={
            image?.media_details?.sizes?.full?.file ||
            image?.slug ||
            getFilename( url )
          }
          allowedTypes={ ALLOWED_MEDIA_TYPES }
          onSelect={ onSelectMedia }
          onError={ onUploadError }
          onReset={ onClearMedia }
          useFeaturedImage={ useFeaturedImage }
          onToggleFeaturedImage={ toggleUseFeaturedImage }
          emptyLabel={ __( 'Add media' ) }
        />
      </ToolsPanelItem>
    </ToolsPanel>
  </InspectorControls>
) : null;
```

Key details:
- The panel is gated on `isSelected` (new prop threaded from `CoverEdit`), not on `!! url || useFeaturedImage`, so it appears for an empty Cover.
- The existing settings panel (position, overlay, dim ratio, etc.) remains behind the `!! url || useFeaturedImage` guard.
- New imports: `getFilename` from `@wordpress/url`, `ALLOWED_MEDIA_TYPES` from `../shared`, and `MediaControl` from `../../utils/media-control`.
- New props added to `CoverInspectorControls`: `onSelectMedia`, `onUploadError`, `toggleUseFeaturedImage`, `onClearMedia`, `isSelected`.
- The `MediaControl` component gains a `useFeaturedImage` / `onToggleFeaturedImage` pair (the "Use featured image" toggle), consistent with the change in PR #82593.
- Browser tests in `packages/block-library/src/cover/test/edit.browser.test.js` were updated: a helper `openSettingsTabIfAvailable()` was added, and the test that previously asserted the Settings tab was absent for an empty Cover now asserts the Settings *heading* (panel content) is absent while the tab itself may exist. The e2e spec in `test/e2e/specs/editor/blocks/cover.spec.js` adds a click on the Settings tab before interacting with the focal point picker.

## Contribution

Opened by @im3dabasia as part of a series of PRs standardising the inspector panel across media-related blocks (Cover, and a companion for another block referenced as #82593). @jasmussen reviewed and approved it as a valid fix for "placeholders in constrained contexts," noting it only needed a code review. @im3dabasia fixed failing tests in a follow-up commit (`cc284c4`) before merge. Co-authors per the merge commit: @talldan, @Mamaduka, @jasmussen. The PR notes use of Claude Code as an AI tool.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
