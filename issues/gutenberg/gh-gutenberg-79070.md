# #79070: Media Editor: Keep the modal skeleton pinned in the editor flow

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`32bfd70`](https://github.com/WordPress/gutenberg/commit/32bfd70fc7a25479c832b5f7a4df48f9757fc53f)
- **Discussion:** [#79070](https://github.com/WordPress/gutenberg/pull/79070) · 3 comments · 0 reactions

## Summary

Fixes a CSS specificity regression in the block editor's media modal where an external experiment's admin bar offset rule incorrectly pushed the internal skeleton down, clipping the fine-rotation ruler. The change anchors the `top: auto` reset to the modal frame using doubled class specificity so it always overrides external offsets, preserving the intended fullscreen editing layout.

## Impact

- **Plugin & Theme Developers / Block Editor Contributors**: No action required. This resolves a UI layout bug in the media editor when the `gutenberg-admin-bar-in-editor` experiment is active.
- **Site Owners / Editors**: No action required. The fix is internal to the block editor frontend and restores correct positioning for image editing in fullscreen mode.
- **Hosting & Platform Teams / Headless & REST Consumers**: No impact. This change affects only Gutenberg's media package CSS and does not touch PHP, database schemas, or REST endpoints.

## Technical details

The update targets `packages/media-editor/src/components/media-editor/style.scss` to enforce a layout reset at a higher specificity than external experiment rules. The diff adds:
```scss
.media-editor-modal & .interface-interface-skeleton.interface-interface-skeleton {
    top: auto;
}
```
The selector leverages SCSS parent referencing (`&`) to resolve to the modal frame, and deliberately duplicates `.interface-interface-skeleton` to outrank offset rules like `body.has-admin-bar-in-editor.is-fullscreen-mode .interface-interface-skeleton`. This ensures the media editor's internal skeleton remains pinned at `top: auto`, preventing experiment-generated admin bar spacing from pushing the canvas down and clipping the fine-rotation ruler beneath it.

## Contribution

Opened and merged by @ramonjd (commit `32bfd70`), with initial regression tracking credited to @andrewserong. During review, an alternative approach anchoring the specificity reset directly to the experiment's `.has-admin-bar-in-editor` class was considered but rejected as fragile for when the experiment graduates. The final implementation instead anchors the rule to the modal frame itself, ensuring resilience against future class changes or experiment removal.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
