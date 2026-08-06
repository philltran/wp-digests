# #81155: PanelColorSettings: Restore the missing space below the panel header

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`
- **Merged:** [`3a96676`](https://github.com/WordPress/gutenberg/commit/3a96676bba69fbc1026707220ae5791a92449619)
- **Discussion:** [#81155](https://github.com/WordPress/gutenberg/pull/81155) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `PanelColorSettings` component now correctly restores the missing vertical spacing below its header. This visual regression was introduced when a prior refactor relocated color panels and inadvertently removed a CSS margin override. The fix aligns the component with the standard `ToolsPanel` grid layout, ensuring consistent spacing for developers using this public API.

## Impact

- **Plugin & theme developers:** No action required. The change is a backward-compatible visual fix.
- **Block editor consumers:** Developers using `PanelColorSettings` in the inspector sidebar will see restored spacing between the header and the first color/gradient control.
- **Hosting & platform teams:** No configuration or migration needed. The adjustment is contained within the block editor package.

## Technical details

The diff modifies `packages/block-editor/src/components/colors-gradients/panel-color-gradient-settings.js` and `packages/block-editor/src/components/colors-gradients/style.scss`. The previous implementation forced a block layout and applied a top margin to the first child, which broke when the parent grid changed. The fix passes `hasInnerWrapper` to `ToolsPanel` and wraps the dropdown and children in a dedicated container. CSS now explicitly spans this container across the grid and removes the row gap.

Before:
```jsx
<ToolsPanel ...>
  <ColorGradientSettingsDropdown ... />
  { !! children && (
    <>
      <Spacer marginY={ 4 } /> { children }
    </>
  ) }
</ToolsPanel>
```

After:
```jsx
<ToolsPanel hasInnerWrapper ...>
  <div className="block-editor-panel-color-gradient-settings__inner-wrapper">
    <ColorGradientSettingsDropdown ... />
    { !! children && (
      <>
        <Spacer marginTop={ 4 } marginBottom={ 0 } /> { children }
      </>
    ) }
  </div>
</ToolsPanel>
```

CSS adjustments remove the legacy `.block-editor-panel-color-gradient-settings.block-editor-panel-color-gradient-settings { display: block; }` override and replace it with grid-spanning rules for the new inner wrapper.

## Contribution

Opened and merged as a direct follow-up to #77279. During review, the author verified that the new wrapper approach correctly preserves the component's ability to render arbitrary children without disrupting `ToolsPanel`'s grid placement. The original `display: block` workaround was explicitly rejected in favor of the standard `hasInnerWrapper` pattern used by sibling color panels.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
