# #81319: Block Editor: Fix Escape from the block toolbar and stop redundant last focus dispatches

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`a370a8b`](https://github.com/WordPress/gutenberg/commit/a370a8b6b707d59e0cec660792311a33b7dcbc08)
- **Discussion:** [#81319](https://github.com/WordPress/gutenberg/pull/81319) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Pressing Escape in the block toolbar now moves focus back to the selected block when no `lastFocus` has been recorded yet, for example after inserting a block from the global inserter and pressing Alt+F10. Previously Escape did nothing and left keyboard users stranded in the toolbar. The PR also stops the block editor from dispatching `lastFocus` updates on every in-canvas `focusout`, which was notifying all store subscribers and re-running every `useSelect` for no reason.

## Impact

- **Plugin & theme developers:** No API changes. Custom blocks keep the existing behavior where focus returns to the element that last held it (e.g. the Image block's Upload button) rather than the block wrapper.
- **Editor users / accessibility:** Keyboard users who insert a block through the global inserter can now get back to the canvas with Alt+F10 then Escape.
- **Performance:** Fewer block editor store updates while moving focus between blocks, so fewer `useSelect` re-runs. The PR reports a +43 B size change in `block-editor/index.min.js`.
- **Action required:** None.

## Technical details

**`navigable-toolbar/index.js`** (`useToolbarFocus`): the Escape `keydown` handler now resolves its target as `getLastFocus()?.current ?? refsMap.get( getSelectedBlockClientId() )`. `refsMap` comes from `useContext( BlockRefs )` (imported from `../provider/block-refs-provider`). Selection is read imperatively inside the handler via `getSelectedBlockClientId` from `unlock( useSelect( blockEditorStore ) )`, so the toolbar stays non-reactive to selection. `preventDefault()` is now called only when a target exists. The effect returns early when `focusEditorOnEscape` is false, and its dependency array now includes `getSelectedBlockClientId` and `refsMap`.

**`writing-flow/use-tab-nav.js`** (`onFocusOut`): previously every `focusout` ran `setLastFocus( { ...getLastFocus(), current: event.target } )`. Now it runs only when `! node.contains( event.relatedTarget )`, i.e. focus actually left the canvas. A null `relatedTarget` (focus moving to another document) also counts as leaving. The spread of the previous value is dropped: `setLastFocus( { current: event.target } )`.

```js
// before
setLastFocus( { ...getLastFocus(), current: event.target } );
// after
if ( ! node.contains( event.relatedTarget ) ) {
	setLastFocus( { current: event.target } );
}
```

**Tests** (`navigable-toolbar.spec.js`): removes the `BlockToolbarUtils` fixture class in favor of direct `pageUtils.pressKeys` and `editor.getFocusOwnerLabel` polling. Adds e2e cases for Escape and Tab returning focus to the Image block's Upload button, and for Escape returning focus to the block when the canvas has never been focused (using a fixed toolbar and the global inserter). The scrollable-toolbar test is rewritten to use `toBeInViewport` with mouse wheel scrolling, and waits for toolbar remount after a viewport change.

## Contribution

Authored by @Mamaduka with Claude assistance, as disclosed in the PR. It extends #61472, which fixed the same stranded-focus problem only for the Shift+Tab path, and preserves the non-reactive selection access from #57140. The only human comment is the author's pre-merge note that the PR has no intended behavior changes beyond the fixes and added e2e coverage. It also says the `Tab order of the block toolbar aligns with visual order` test is still somewhat flaky, though less so than on trunk, with further debugging planned as a follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
