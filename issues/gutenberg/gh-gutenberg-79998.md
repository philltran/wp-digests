# #79998: Responsive editing: Hide block toolbar slots when editing a responsive style state

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Feature] Blocks`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`bf74f45`](https://github.com/WordPress/gutenberg/commit/bf74f4531688e1c2e64465a71709e2b225ec0cdf)
- **Discussion:** [#79998](https://github.com/WordPress/gutenberg/pull/79998) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

When editing a block for a specific viewport, the block editor now hides non-essential toolbar slots. This prevents misleading UI since toolbar controls currently do not apply to responsive style states, unlike inspector controls.

## Impact

- **Block Editor & Plugin Developers**: Toolbar visibility is now dynamically gated during responsive style editing. Components or custom `BlockControls` that assume persistent slot presence in this mode will observe hidden controls.
- **Site Owners/Editors**: Will see a reduced toolbar (e.g., alignment, color, background) when adjusting styles for tablet/mobile breakpoints. Controls return to normal when switching viewports or disabling responsive editing.
- **Migration/Action**: No code changes required. This is an internal editor UX adjustment that may affect custom block implementations relying on standard slot behavior during style state edits.

## Technical details

In `packages/block-editor/src/components/block-toolbar/index.js`, `PrivateBlockToolbar` now imports `hasViewportBlockStyleState` from `../../hooks/block-style-state`. It pulls `getSelectedBlockStyleState` and `isResponsiveEditing` from the `blockEditorStore` selector. A new condition evaluates `_isEditingResponsiveStyleState = isResponsiveEditing() && hasViewportBlockStyleState( getSelectedBlockStyleState( selectedBlockClientId ) )`, which gates the existing `showSlots` prop: `showSlots: ! _isZoomOut && ! _isEditingResponsiveStyleState`. This effectively suppresses toolbar slots while responsive editing is active and a block holds a viewport-specific style state.

## Contribution

Opened and merged by @talldan with co-authors @t-hamano, @andrewserong, and @Mamaduka. Reviewers noted that exposing non-functional controls during responsive edits was confusing; the merge consensus favored hiding slots temporarily until full responsive editing support lands, framing it as an intentional UI limitation rather than a regression.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
