# #79981: Tabs: Fix active tab switching from a stale inner-block selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`cddb1e1`](https://github.com/WordPress/gutenberg/commit/cddb1e1547dbb2f6a0b452a4b0658205b9dfb5e1)
- **Discussion:** [#79981](https://github.com/WordPress/gutenberg/pull/79981) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a selection-state race condition in the `core/tabs` block where clicking another tab fails to switch panels if an inner block in the active panel remains selected. The underlying bug occurred because a sync effect in the tab-panel block detected a stale inner-block selection and overwrote the `editorActiveTabIndex`. Wrapping the update in a synchronous batch prevents the stale state from interfering, ensuring tab switching behaves predictably during editing.

## Impact

- **Block & Theme Developers**: No public API changes or breaking updates. The fix operates entirely within core block editor internals.
- **Plugin/Custom Block Developers**: If you extend `core/tabs` or build blocks that interact with its selection state, tab switching will now correctly persist without reverting due to internal race conditions.
- **Editors & Site Owners**: Resolves a frustrating UX bug where clicking a new tab’s header would not activate it if the previous panel’s content was still selected or cursor-focused.
- **Action Required**: None. This is a self-contained editor bug fix with no migration steps, configuration changes, or code updates needed.

## Technical details

The primary change occurs in `packages/block-library/src/tab-list/edit.js`. The `Edit` component now imports `useRegistry` from `@wordpress/data` and adds `selectBlock` to the `useDispatch( blockEditorStore )` destructure. Inside `selectTabPanel`, the existing `updateBlockAttributes` call is wrapped alongside `selectBlock( clientId )` within `registry.batch()`. Previously, these operations executed sequentially; batching them guarantees the tab-list selection state updates before the tab-panel’s sync effect runs, preventing it from observing a stale selection and reverting `editorActiveTabIndex`. Additionally, a new e2e test suite was added at `test/e2e/specs/editor/blocks/tabs.spec.js` to cover keyboard navigation, tab switching with active inner selections, and undo/redo constraints for non-persistent index changes.

## Contribution

Open and merged by @t-hamano with code review from @mamaduka and @mcsf. The PR addresses a stale state regression identified in Gutenberg #79848. Review focused on implementing `registry.batch()` to synchronize selection and attribute updates, eliminating the race condition. E2E coverage was added during review to validate keyboard interactions, inner-block selection handling, and undo history behavior for tab modifications.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
