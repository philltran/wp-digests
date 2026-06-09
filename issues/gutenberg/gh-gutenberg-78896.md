# #78896: Media Editor: refactor modal layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Feature] Media`
- **Merged:** [`e467860`](https://github.com/WordPress/gutenberg/commit/e46786098261710fb50f7ef98b72c9af48506cf9)
- **Discussion:** [#78896](https://github.com/WordPress/gutenberg/pull/78896) · 18 comments · 1 reactions

## Summary

The Media Editor modal layout has been refactored to reorganize editing controls for improved spatial efficiency across viewport sizes. On wide viewports, crop controls now group in a right-hand panel while the fine-rotation ruler remains anchored beneath the canvas. The crop surface overlay receives a lighter visual treatment, and the existing Details tab is preserved to maintain the extension surface for metadata, plugin panels, and AI features.

## Impact

- **Block & Plugin Developers**: No breaking changes to the media editor API, hooks, or block.json schemas. UI layout adjustments are internal to the Gutenberg editor surface.
- **Site Owners & Editors**: Control reflow improves usability on large screens; rotation ruler stays locked to canvas width during resizing. Undo/redo state indicators now correctly display accent colors.
- **Headless & REST Consumers**: Zero impact. No endpoint, schema, or authentication changes.
- **Action Required**: None. This is a pure UI/layout refinement with no migration steps or configuration changes needed.

## Technical details

The implementation adjusts the modal's CSS and component structure to enforce a responsive split layout. On wide viewports, editing controls (Rotate, Flip, Zoom, Aspect Ratio) move to a right panel; the fine-rotation ruler stays locked to the canvas width inside the grey panel at all viewport sizes. The crop surface applies a lighter grey wash overlay with a dark outline and white handles featuring blue borders. Importantly, the `Tabs` component retains its legacy `Tabs.Context` bridge across Slot/Fill boundaries to keep metadata rendering functional in the Details tab. Follow-ups note that migrating these tabs to `@wordpress/ui` is deferred due to Base UI's lack of public context exposure, which would require restructuring the sidebar under a single `Tabs.Root` or adding cross-subtree state support.

## Contribution

Opened by @ramonjd to address layout proposals from issue #73771. Review confirmed the decision to retain the Details tab over an earlier proposal to drop it, citing its critical role as the extension surface for metadata and plugin panels. The PR was merged after manual testing verified responsive reflow, RTL compatibility, control state rendering, and zoom slider thumb clipping fixes. Explicit follow-ups for mobile footer controls (zoom +/- and aspect ratio) and a component migration were logged but excluded from this merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
