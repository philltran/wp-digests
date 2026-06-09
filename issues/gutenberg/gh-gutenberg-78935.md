# #78935: Media Editor: Add aspect ratio control to mobile toolbar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`3556824`](https://github.com/WordPress/gutenberg/commit/3556824395809d72d1f66fc4ff973a354f6752ff)
- **Discussion:** [#78935](https://github.com/WordPress/gutenberg/pull/78935) · 3 comments · 1 reactions

## Summary

Introduces an aspect ratio control to the media editor's mobile toolbar as an icon dropdown. This change closes a viewport parity gap by allowing editors to adjust image proportions directly in the narrow footer toolbar, alongside existing rotate and flip controls, matching the functionality previously only available in the desktop sidebar crop panel.

## Impact

- **Block & theme developers**: No code changes or migration steps required. The modification is isolated to the Gutenberg editor frontend UI state and rendering logic.
- **Site owners / Editors**: Mobile image editing workflows now include aspect ratio presets in the bottom toolbar, streamlining cropping tasks on touch devices.
- **No action required**. The feature ships automatically with the updated block editor build.

## Technical details

- Extends the flat media editor transform toolbar to render an optional aspect ratio dropdown icon.
- Wires the narrow/mobile footer toolbar (`< 782px breakpoint`) directly into the existing crop options state, ensuring immediate synchronization with the image transform pipeline.
- Preserves the labeled sidebar crop panel behavior for wider viewports, avoiding UI duplication or layout regression.
- The change operates entirely at the UI/state layer; no new REST routes, `block.json` fields, PHP filters, or public JavaScript APIs are introduced.

## Contribution

Open and merged by @ramonjd, with co-authorship from @andrewserong per the final merge commit. Labeled as a `[Type] Enhancement` and `[Feature] Media`, it serves as a direct follow-up to an earlier PR that added aspect ratio controls to the desktop editor toolbar. Review was lightweight and focused on visual parity and viewport testing; no alternative approaches or significant design debates were recorded in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
