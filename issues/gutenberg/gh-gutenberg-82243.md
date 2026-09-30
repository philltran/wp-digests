# #82243: Meta Boxes: Move meta box markup with moveBefore to keep classic editors alive

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Feature] Meta Boxes`, `[Package] Edit Post`
- **Merged:** [`5a56827`](https://github.com/WordPress/gutenberg/commit/5a56827f47c6dcb37e55cd9df8264740910a078d)
- **Discussion:** [#82243](https://github.com/WordPress/gutenberg/pull/82243) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`MetaBoxesArea` in the Gutenberg post editor now relocates the meta box form with `Element.moveBefore()` instead of `appendChild()`, and does so from a layout effect rather than a passive one. `appendChild` resets an `<iframe>`'s browsing context, which detached TinyMCE from its document and left classic editors in meta boxes (e.g. `wp_editor()` in a postbox) broken. `moveBefore` preserves subtree state, so the iframe isn't reloaded. Browsers without `moveBefore` (Safari) fall back to the old behavior and still exhibit the bug.

## Impact

- **Plugin developers with TinyMCE/`wp_editor()` in meta boxes:** classic editors in meta boxes should stay functional in Chromium and Firefox, including after toggling Distraction free mode (the PR's reproduction case). No code changes required.
- **Safari users:** not fixed; the `appendChild` fallback keeps the previous failure mode. The discussion explicitly acknowledges this.
- **Site owners / hosting / headless:** no action required. No new APIs, hooks, or REST changes.

## Technical details

Changes are in `packages/edit-post/src/components/meta-boxes/meta-boxes-area/index.jsx`:

- Adds a local helper `move( parent, node )` that calls `parent.moveBefore( node, null )` when `parent.moveBefore` exists and both `parent` and `node` are `isConnected`; otherwise it falls back to `parent.appendChild( node )`.
- Swaps `useEffect` for `useLayoutEffect` (imported from `@wordpress/element`). The cleanup of a passive effect runs after React has detached the subtree, at which point `moveBefore` throws `HierarchyRequestError`; a layout effect's cleanup runs while nodes are still connected.
- Both the mount move (into `container.current`) and the cleanup move (back into `#metaboxes`) now go through `move()`.

```js
// before
container.current.appendChild( formRef.current );
// after
move( container.current, formRef.current );
```

The PR's rationale: `class-wp-editor.php` defers classic editor init in postboxes until `document.readyState === 'complete'`, but Gutenberg mounts after that, so the move lands during or after `tinymce.init()`. Landing inside the init window leaves `editor.initialized` unset; landing afterward leaves the editor bound to a detached document. Neither is recoverable because `switchEditor()` returns early once `tinymce.get( id )` exists. A `CHANGELOG.md` entry was added for `@wordpress/edit-post`.

## Contribution

@Mamaduka authored the fix, noting `moveBefore` isn't Baseline but is the cleanest option, and asked @jsnajdr for review. Both agreed there was no better alternative, and that since no user reports of the issue had surfaced (only e2e test flakiness), it was safe to ship as a progressive fix while accepting the remaining Safari failure. The PR notes it was assisted by Claude.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
