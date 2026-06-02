# #78384: Responsive block instance styles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Feature] Blocks`, `[Package] Editor`, `[Package] Block editor`
- **Merged:** [`5bfdf7b`](https://github.com/WordPress/gutenberg/commit/5bfdf7b33d408510ac572a810dad57daec9c3164)
- **Discussion:** [#78384](https://github.com/WordPress/gutenberg/pull/78384) · 14 comments · 5 reactions

## Summary

PR #78384 adds mobile and tablet responsive states to the per-block style system, letting editors set different visual styles for each viewport on any block that supports styles — without requiring a new `block.json` declaration. Selecting a responsive state in the block inspector's state dropdown simultaneously switches the editor canvas to the corresponding viewport preview so changes are immediately visible on the canvas. The implementation follows the same pattern as the existing pseudo-state (hover/focus) system introduced in #77817, and responsive states can be combined with pseudo-states (e.g., mobile + hover). Inspector controls are filtered to show only those relevant to the active state.

## Impact

**Content editors / site builders**
- New "Mobile" and "Tablet" options appear in the block state dropdown for any block that supports styles; no theme or plugin change required to unlock them.
- Switching states changes the canvas viewport, so responsive tweaks are previewed in-context.

**Plugin & theme developers**
- No new `block.json` `supports` key is required at this stage; the feature activates automatically for blocks that support styles.
- Responsive and pseudo-states can be stacked; verify any custom style-generation code that reads the editor state store handles the new combined-state shape.
- A UX edge case was flagged in review (@tellthemachines): the canvas resets to desktop when the user selects a different block, regardless of whether the responsive preview was triggered by a block state selection or by the user manually. This behaviour was noted but whether it was resolved before merge is not confirmed in the available discussion.

**No breaking changes or deprecated APIs identified in the provided material.**

## Technical details

The block inspector's state dropdown gains "Mobile" and "Tablet" entries alongside existing pseudo-states. The `@wordpress/block-editor` state store value for active block state is updated to support combinations of responsive and pseudo states simultaneously (e.g., a block can be in `mobile` + `hover` at once).

Style calculation is routed through the existing `state` supports hook — no new block supports key is introduced. When a responsive state is active the editor canvas viewport is programmatically switched to the matching breakpoint preview (mobile → mobile viewport, tablet → tablet viewport).

Bundle impact from the build artefacts:
- `build/scripts/block-editor/index.min.js` +768 B (+0.22 %)
- `build/scripts/editor/index.min.js` +231 B (+0.05 %)

No diff was available in the source material; the above is derived from the PR description and size report.

## Contribution

Opened and authored by @talldan, who noted use of Codex/OpenCode for initial code generation with a manual review pass. Merged at commit `5bfdf7b`. @tellthemachines raised a UX concern during review — the canvas reverts to the desktop viewport when another block is selected, potentially resetting a preview state the user had set manually. The discussion excerpt ends before a resolution to that concern is recorded. Additional review participants visible in the co-author list include @ramonjd, @andrewserong, @kjellr, @jameskoster, and @jasmussen, among others.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
