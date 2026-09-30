# #81456: Writing flow: stop the caret on containers that do not merge with the text flow

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`175d53c`](https://github.com/WordPress/gutenberg/commit/175d53c3e18f033529c143740d40db1dc2e71583)
- **Discussion:** [#81456](https://github.com/WordPress/gutenberg/pull/81456) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Arrow-key navigation in the block editor once again stops on container blocks such as group, columns, cover, and gallery. A wrapper is now skipped only if its block type merges with the text flow, meaning it defines a `merge` function or has the `__experimentalOnMerge` support. List, list item, and quote stay transparent. The group block also loses `__experimentalOnMerge`, so pressing Backspace at the start of the block after a group selects the group rather than pulling that block inside it.

## Impact

- **Site owners / editors:** Arrowing through content now lands on group, columns, cover, gallery and similar containers, which restores a keyboard path to select them. Lists and quotes behave as before, with no wrapper selection. Backspace after a group selects the group and merges nothing.
- **Plugin & theme developers:** Custom container blocks are now selectable via arrow keys unless they declare a `merge` function or `__experimentalOnMerge` support. Blocks that declare either are treated as part of the text flow and remain skipped. Check custom wrapper blocks if you relied on the previous skip-all-wrappers navigation.
- **Test authors:** E2E tests that assumed arrow navigation skipped wrappers may break. The Gutenberg tests for columns, writing flow, and toolbar roving tabindex were updated for this.
- **Breaking changes:** No public API is removed or deprecated. The change is behavioral only.

## Technical details

**`packages/block-editor/src/components/writing-flow/use-arrow-nav.js`**: `isTabCandidate()` in `getClosestTabbable` changed.

- New early return: a node is skipped when it has exactly one child, that child is in the same block (`isInSameBlock`, newly imported from `../../utils/dom`), and the child has `contenteditable="true"`.
- The nested-focusable skip for non-contenteditable block wrappers is now gated on the block type. It applies only if `getBlockType( node.getAttribute( 'data-type' ) )` exists and has `merge` or `hasBlockSupport( blockType, '__experimentalOnMerge' )`. Otherwise the wrapper stays a tab candidate, so the caret stops on it.
- The diff removes the earlier comment referencing PR #77474 and the TODO about `focus.tabbable`.
- `getBlockType` and `hasBlockSupport` are newly imported from `@wordpress/blocks`.

**`packages/block-library/src/group/block.json`**: removes `"__experimentalOnMerge": true` from `supports`. Group thereby becomes both a caret stop and a non-merging container, since the same trait now drives both behaviors.

```diff
 "__experimentalOnEnter": true,
-"__experimentalOnMerge": true,
 "__experimentalSettings": true,
```

**Tests**: the e2e test `can merge into group with Backspace` is replaced by one asserting the group is selected and the blocks are unchanged, and its two snapshot files are deleted. The columns, writing-flow, and toolbar-roving-tabindex specs are updated to expect navigation through the `core/column` and `core/columns` wrappers and the `Block: Table` wrapper. No hooks, REST schema, or DB changes.

## Contribution

The PR description traces the history: #42780 introduced generic wrapper merging for quote and list, with group swept in; #53508 made it opt-in via `__experimentalOnMerge` after Columns broke; #75141 made arrow navigation skip all non-empty wrappers. Feedback in #76568 and #78955 pushed back on that, converging on list and quote as "ephemeral containers" and everything else as hard boundaries. This PR partially restores pre-#75141 navigation. It is authored by @ellatrix and the description notes it was generated with Claude Code. The discussion on the PR consists only of bot comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
