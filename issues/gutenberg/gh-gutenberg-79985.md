# #79985: GIF block variation: Remove icons from "Display as" toolbar buttons and only show when block can be inserted

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Feature] Media`, `[Package] Block library`, `[Feature] Client Side Media`
- **Merged:** [`668ad53`](https://github.com/WordPress/gutenberg/commit/668ad53841814765265511f31dd898914979c378)
- **Discussion:** [#79985](https://github.com/WordPress/gutenberg/pull/79985) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This merge refines the animated GIF-to-video conversion controls in the block editor by removing toolbar icons and replacing hardcoded gallery checks with dynamic `canInsertBlockType` validation. Previously, the "Display as video" and "Display as GIF" buttons displayed both text and icons, diverging from standard toolbar conventions, and relied on a hardcoded check that failed to respect editor preferences or custom allowed-block configurations. The update aligns UI styling and enforces proper block-insertion rules so conversion controls remain hidden when the target block cannot be placed in the current context.

## Impact

- **Plugin & theme developers**: No breaking changes or required migrations. Developers extending image/video blocks should note the shift to `canInsertBlockType` for validating target insertability instead of hardcoding parent block names.
- **Site owners / editors**: Improved UI consistency in the Image and Video block toolbars; conversion buttons now correctly respect disabled states and Preferences panel settings.
- **Hosting & platform / headless & REST consumers**: No direct impact.

## Technical details

In `packages/block-library/src/image/animated-gif-convert-control.js` and `packages/block-library/src/video/gif-restore-control.js`, the hardcoded gallery bypass (`getBlockName( rootClientId ) === 'core/gallery'`) was replaced with dynamic checks using `canInsertBlockType( 'core/video', getBlockRootClientId( clientId ) )` and `canInsertBlockType( 'core/image', ... )` respectively. Both components strip their `icon` props from `<ToolbarButton>`.

In `packages/block-library/src/image/image.js`, the previously inlined media controls were extracted into a unified `mediaControls` JSX array to preserve consistent rendering order alongside `MediaReplaceFlow`:

```jsx
// Before
const mediaReplaceFlow = isSingleSelected && ... <BlockControls>...</BlockControls>;

// After
const mediaControls = isSingleSelected && ... (
  <>
    <BlockControls>...<MediaReplaceFlow /></BlockControls>
    <AnimatedGifConvertControl />
  </>
);
```

Tests for `animated-gif-convert-control.js` were updated to mock `canInsertBlockType` instead of simulating `rootBlockName`. This is an internal block-editor UI refinement; no public APIs, hooks, or REST endpoints were added or modified.

## Contribution

Open by @andrewserong as a minimally scoped fix tied to broader work in #79787. Reviewer @Mamaduka suggested consolidating control rendering into a shared `mediaControls` variable and reordering it next to `MediaReplaceFlow` to maintain visual consistency across the image and video blocks. The final diff reflects these refinements, stripping icons and implementing dynamic insertion checks before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
