# #79181: Template Part: Remove restriction on tabs / inspector fills

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `[Block] Template Part`, `Backport to WP Minor Release`
- **Merged:** [`667b451`](https://github.com/WordPress/gutenberg/commit/667b4517ebc90ee666b284cf93efe1650e350d9f)
- **Discussion:** [#79181](https://github.com/WordPress/gutenberg/pull/79181) · 7 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

A regression in the Block Editor restricted `core/template-part` from rendering custom inspector fills and style tabs added by plugin authors or site extenders. This PR relaxes the `InspectorControlsFill` and `StylesTab` gating logic, restoring the historical capability for extenders to inject arbitrary block inspector groups into the Template Part's inspector without being filtered out by core's group whitelists.

## Impact

- **Plugin/Theme Developers**: Restores the ability to use `InspectorControls.Slot` and custom style fills within the Template Part editor. Any developer relying on this extensibility will see their added controls reappear in 7.0.1+.
- **Site Owners/Editors**: No direct change; the UI only exposes groups if an extender populates them.
- **Core Maintenance**: Removes the `TEMPLATE_PART_GROUPS` whitelist from `InspectorControlsFill`, simplifying the rendering logic to rely primarily on `isTemplatePart` identity.

## Technical details

The diff modifies two components in `packages/block-editor/src/components/inspector-controls/`:
1. `fill.js`: Deletes the `TEMPLATE_PART_GROUPS` constant and replaces the restrictive filtering logic:
```diff
- const canShowGroup = ( isTemplatePart && isTemplatePartGroup ) || isPatternEditingGroup;
+ const canShowGroup = isTemplatePart || isPatternEditingGroup;
```
This bypasses the group whitelist check entirely when `context.name === 'core/template-part'`.
2. `styles-tab.js`: Updates the `StylesTab` conditional to explicitly include `core/template-part`:
```diff
- { ! isSectionBlock && (
+ { ( ! isSectionBlock || blockName === 'core/template-part' ) && (
```
This ensures the styles tab container renders for template parts, allowing underlying style fills to display.

## Contribution

Opened by @talldan as a direct fix for issue #79152, identifying that prior restrictions in PRs #71714 and #74793 were unnecessary for Template Parts. After rapid review and agreement from @ramonjd and @webmandesign that the historical extender behavior should be restored, @cbravobernal cherry-picked the change to the `wp/7.0` branch targeting the 7.0.1 point release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
