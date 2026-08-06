# #81142: Video: Hide settings for the GIF variation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @yogeshbhutkar
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Video`, `Backported to WP Core`, `[Feature] Client Side Media`
- **Merged:** [`a5e3bb4`](https://github.com/WordPress/gutenberg/commit/a5e3bb48d79181eb83a85e44ee485e73ed087044)
- **Discussion:** [#81142](https://github.com/WordPress/gutenberg/pull/81142) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Video block now hides its sidebar settings panel when the GIF variation is active. Previously, editors could accidentally modify playback or poster attributes in the GIF variation, which would break the expected animated behavior. This change prevents configuration drift by conditionally rendering the inspector controls only for standard video blocks.

## Impact

- **Block & theme developers:** No code changes required. The block's internal rendering logic now conditionally hides `InspectorControls` based on the `isGif` flag.
- **Site owners & editors:** The Video block's settings sidebar will no longer appear when the block is set to the GIF variation, preventing accidental changes to playback attributes.
- **No action required** for existing sites or custom blocks using the Video block, as this is a client-side UI adjustment with no API changes.

## Technical details

The change modifies `packages/block-library/src/video/edit.js` by wrapping the `<InspectorControls>` component in a conditional render. The `isGif` flag is already computed in the component based on the block's attributes (`autoplay`, `controls`, `loop`, `muted`, `playsInline`). When `isGif` is true, the `<ToolsPanel>` containing `<VideoCommonSettings>` and `<PosterImage>` is omitted from the DOM.

**Before:**
```jsx
<InspectorControls>
  <ToolsPanel label={ __( 'Settings' ) } ...>
    <VideoCommonSettings setAttributes={ setAttributes } attributes={ attributes } />
    <PosterImage poster={ poster } onChange={ ... } />
  </ToolsPanel>
</InspectorControls>
```

**After:**
```jsx
{ ! isGif && (
  <InspectorControls>
    <ToolsPanel label={ __( 'Settings' ) } ...>
      <VideoCommonSettings setAttributes={ setAttributes } attributes={ attributes } />
      <PosterImage poster={ poster } onChange={ ... } />
    </ToolsPanel>
  </InspectorControls>
) }
```

## Contribution

Opened by @yogeshbhutkar to address issue #81125, the PR was reviewed by @Mamaduka, @annezazu, and @adamsilverstein. @Mamaduka noted it functioned more as a bug fix since accidental setting adjustments shouldn't override the GIF variation's behavior. After a brief UI rendering quirk was flagged during playground testing and deemed non-reproducible, @adamsilverstein merged the PR to enable backporting to the `wp/7.1` branch before RC1.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
