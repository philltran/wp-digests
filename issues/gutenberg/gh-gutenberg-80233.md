# #80233: Responsive Editing: support editing pattern styles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Feature] Patterns`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`90542b4`](https://github.com/WordPress/gutenberg/commit/90542b47a8e6d7d6269c9131b1d5e5461a17c79e)
- **Discussion:** [#80233](https://github.com/WordPress/gutenberg/pull/80233) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

When responsive editing is enabled and a tablet or mobile viewport is selected, the block inspector previously hid all style controls for unsynced patterns. This change restores those controls by conditionally rendering the `SectionStyleControls` component within `StyleStateInspectorSlots` when a section block is active. The fix ensures pattern styling remains accessible across responsive viewports without altering existing block editor behavior.

## Impact

- **Plugin & theme developers**: No code changes required. The block inspector behavior is internal to the editor; developers will simply see restored controls when testing responsive editing with patterns.
- **Site owners**: Pattern styling controls are restored in the block inspector when switching to tablet or mobile viewports during responsive editing.
- **No action required** for existing codebases, configurations, or third-party extensions.

## Technical details

The diff modifies `packages/block-editor/src/components/block-inspector/index.js` and `packages/block-editor/src/components/inspector-controls-tabs/styles-tab.js`. Previously, `StyleStateInspectorSlots` unconditionally rendered a flat list of `InspectorControls.Slot` groups. The component now accepts `clientId`, `contentClientIds`, and `isSectionBlock` props. It evaluates `const showSectionStyleControls = isSectionBlock && blockName !== 'core/template-part';` to conditionally render `SectionStyleControls` wrapped in a `BlockStyleStateProvider`. When the condition is false, it falls back to the original slot groups. `BlockInspectorSingleBlock` now passes these props when `isEditingStyleState` is true, and `SectionStyleControls` is exported from `styles-tab.js`.

**Before**
```jsx
// StyleStateInspectorSlots always rendered InspectorControls.Slot groups
<InspectorControls.Slot group="typography" label={ __( 'Typography' ) } />
// ... other slots
```

**After**
```jsx
const showSectionStyleControls = isSectionBlock && blockName !== 'core/template-part';
{ showSectionStyleControls && (
  <BlockStyleStateProvider value={ selectedBlockStyleState }>
    <SectionStyleControls blockName={ blockName } clientId={ clientId } contentClientIds={ contentClientIds } />
  </BlockStyleStateProvider>
) }
{ ! showSectionStyleControls && (
  <>
    <InspectorControls.Slot group="typography" label={ __( 'Typography' ) } />
    // ... other slots
  </>
) }
```

## Contribution

Opened by @talldan, the pull request was co-authored by @ramonjd, @tellthemachines, and @noruzzamans. After initial review and testing confirmation, @t-hamano and @tellthemachines coordinated a backport to the `wp/7.1` branch and Gutenberg 23.6 RC to align with the upcoming release cycle. The change was merged as `90542b4` with no significant design debates noted in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
