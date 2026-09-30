# #83196: Video: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Video`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`4189ebb`](https://github.com/WordPress/gutenberg/commit/4189ebb45cd88e832569688e99b6c66be09efd51)
- **Discussion:** [#83196](https://github.com/WordPress/gutenberg/pull/83196) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Video block now supports the `shadow` block support, allowing shadows to be applied via Global Styles or the block inspector. The shadow is routed to the inner `<video>` element rather than the wrapping `<figure>`, so it frames only the video and never the caption. This brings the Video block in line with other media blocks (Image, Cover) as part of the ongoing design-tools consistency effort tracked in #43241.

## Impact

- **Site builders / theme developers:** The Video block now exposes a Shadow control in Global Styles (Blocks → Video → Shadow) and in the block inspector. The CSS selector for the shadow is `.wp-block-video video`, not `.wp-block-video` — any custom CSS targeting the figure for shadow purposes will not conflict, but custom overrides should target the `video` element.
- **Plugin & theme developers:** No code changes required. The `shadow` support uses `__experimentalSkipSerialization`, so no new attributes are serialized into post content. Existing saved Video blocks validate unchanged; no deprecation or migration is needed.
- **No action required** for existing sites. The saved markup only changes when a shadow is actually set on a block.

## Technical details

Three functional files change:

**`packages/block-library/src/video/block.json`** — adds two keys:

```json
"supports": {
  "shadow": {
    "__experimentalSkipSerialization": true
  }
},
"selectors": {
  "shadow": ".wp-block-video video"
}
```

`__experimentalSkipSerialization` prevents the shadow class/style from being written onto the `<figure>` element. The `selectors` entry tells Global Styles to emit the shadow CSS against `.wp-block-video video` instead of the default block wrapper.

**`packages/block-library/src/video/save.jsx`** and **`edit.jsx`** — both import `__experimentalGetShadowClassesAndStyles` from `@wordpress/block-editor` and build a combined `videoStyle` object:

```jsx
const videoStyle = {
  ...( aspectRatio && { aspectRatio } ),
  ...getShadowClassesAndStyles( attributes ).style,
};
```

This replaces the previous `style={ aspectRatio ? { aspectRatio } : undefined }` on the `<video>` element with `style={ Object.keys( videoStyle ).length ? videoStyle : undefined }`, so the shadow inline style and the aspect-ratio style coexist on the same element.

The `core__video` fixture tests pass unchanged because no shadow is set in the fixture, confirming backward compatibility.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work (related to #43241). The PR was implemented with the assistance of a Claude Code agent, as noted in the description. @ramonjd is credited as a co-author in the merge commit. The discussion is minimal (2 comments, both from the GitHub Actions bot reporting performance metrics and a flaky test), with no design debate or alternative approaches recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
