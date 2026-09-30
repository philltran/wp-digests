# #81659: List View: Make focus on mount opt in

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Editor`, `[Feature] List View`, `[Package] Block editor`, `[Package] Edit Widgets`
- **Merged:** [`090b2ac`](https://github.com/WordPress/gutenberg/commit/090b2acc048dbbd8010a6a861fec63dd557dd1fa)
- **Discussion:** [#81659](https://github.com/WordPress/gutenberg/pull/81659) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor's `ListView` no longer moves focus to the selected block every time its tree grid mounts. Focus-on-mount is now opt-in via a new `focusOnMount` prop, which only the post/site editor and widgets List View sidebars pass. This fixes List Views that mount as a side effect of block selection (e.g. the one in the block inspector, and the Navigation block's inspector list that remounts on every expand) pulling focus out of the editor canvas.

## Impact

- **Plugin & theme developers using `ListView`**: The component is exported as `__experimentalListView` from `@wordpress/block-editor`. If you render it and relied on it focusing the selected block on mount, add `focusOnMount`. Without it, the list no longer takes focus.
- **Editor users**: Blocks whose inspector renders a List View (e.g. Navigation) no longer steal focus from the canvas when selected or when the list remounts. The List View sidebar (toolbar button or `access+o`) still focuses the selected block, or the first block if nothing is selected.
- **Custom sidebars wrapping `ListView`**: Anyone copying the core sidebar pattern with `useFocusOnMount( 'firstElement' )` on a container should drop it and pass `focusOnMount`, since the List View now owns focus placement.
- No action required for code that doesn't depend on List View grabbing focus at mount.

## Technical details

**`packages/block-editor/src/components/list-view/index.js`**
- Adds `focusOnMount = false` prop (documented in the JSDoc).
- Replaces the `focusSelectedBlock` callback ref with `focusOnMountRef`, built with `useRefEffect` from `@wordpress/compose`, and still merged into `treeGridRef`. It returns early when `focusOnMount` is false.
- When enabled, it creates an `AbortController`. It focuses the first selected block via `focusListItem( clientId, node, controller.signal )` and aborts on completion.
- A one-time `focusin` listener on `node.ownerDocument` aborts the controller, so anything else that takes focus first wins.
- With no selection, a `setTimeout( …, 0 )` queries `[role=row][data-block]` and focuses the first row. The code comments say this defers until the opening click is over and avoids focusing inside a still-dirty commit.
- The effect cleanup clears the timer and aborts.

**`list-view/utils.js`**: `focusListItem( focusClientId, treeGridElement, signal )` gains an optional `AbortSignal`. On abort it clears the 3-second timeout, disconnects the `MutationObserver`, and resolves `null`. The `.then` also returns early if `signal.aborted`. This stops the wait for a not-yet-rendered row (nested blocks needing ancestors expanded) from pulling focus back after unmount or after the user moves focus elsewhere.

**Sidebars**: `packages/editor/.../list-view-sidebar/index.js` and `packages/edit-widgets/.../secondary-sidebar/list-view-sidebar.js` remove `useFocusOnMount( 'firstElement' )` and pass `<ListView focusOnMount />`. The PR states this also removes a race between the two focus mechanisms, which depended on whether the selected row rendered in the first commit.

```jsx
// Before: focus always taken on mount
<ListView dropZoneElement={ dropZoneElement } />

// After: opt in
<ListView dropZoneElement={ dropZoneElement } focusOnMount />
```

Also adds a `CHANGELOG.md` entry and an e2e test in `list-view.spec.js` asserting the first block's link is focused when opened with `access+o` after deselecting.

## Contribution

Authored by @Mamaduka, with testing by @andrewserong; the PR discloses it was assisted by Claude. The record shows only bot comments and a thank-you, so no design debate is documented. A flaky `writing-flow.spec.js` e2e test was reported but passed on retry.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
