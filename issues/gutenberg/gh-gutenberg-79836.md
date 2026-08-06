# #79836: Fix/79643 rtc awareness mixed block selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @giteshsarvaiya
- **Labels:** `[Type] Bug`, `[Package] Core data`, `[Package] Editor`, `First-time Contributor`, `[Feature] Real-time Collaboration`
- **Merged:** [`09e1e7a`](https://github.com/WordPress/gutenberg/commit/09e1e7ada3af460cdb679d60246935868519f527)
- **Discussion:** [#79836](https://github.com/WordPress/gutenberg/pull/79836) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Real-time collaboration (RTC) awareness previously failed to render selection overlays when a user selected across mixed block types (e.g., paragraph to image). The fix introduces a `SelectionEndpoint` discriminated union to independently resolve text-based and whole-block endpoints, enabling the collaborators overlay to correctly display semi-transparent fills for text blocks and CSS outlines for non-text blocks. This restores accurate remote selection visibility for multi-block selections, including `Ctrl+A` and keyboard navigation.

## Impact

- **Plugin & theme developers:** No public API changes. Internally, `SelectionInMultipleBlocks` in `@wordpress/core-data` replaced `cursorStartPosition` and `cursorEndPosition` with `startEndpoint` and `endEndpoint`, but this type is not exposed in the public plugin/theme API.
- **Hosting & platform teams:** No configuration changes required.
- **End users:** Remote selection overlays now render correctly across mixed block types, including proper handling of images, spacers, and nested list structures.
- **Action required:** None.

## Technical details

The change restructures how multi-block selections are serialized and rendered in the RTC awareness system. In `packages/core-data/src/types.ts`, `SelectionInMultipleBlocks` now carries `startEndpoint: SelectionEndpoint` and `endEndpoint: SelectionEndpoint` instead of raw `CursorPosition` objects. The `SelectionEndpoint` discriminated union distinguishes between `SelectionType.Cursor` (anchored to a `Y.Text` relative position) and `SelectionType.WholeBlock` (anchored to a block's slot in its parent `Y.Array`).

In `packages/core-data/src/utils/crdt-user-selections.ts`, `getSelectionState` now calls `getSelectionEndpoint` for each side, falling back to block-slot anchoring when no character offset exists. The overlay rendering in `packages/editor/src/components/collaborators-overlay/compute-selection.ts` was refactored: `computeTextSelection` now delegates to `computeMultiBlockOverlayRects` for multi-block cases, applying a two-track visual strategy. Text blocks (`innerText.trim()` is non-empty) receive a 15% opacity semi-transparent fill via `selectionRects`, while non-text blocks (images, spacers) receive a CSS `is-collaborator-selected` outline. The `getBlocksBetween` helper was updated to skip descendant blocks, and a new `blockContainerOf` helper promotes inner blocks (e.g., `core/list-item`) to their nearest `[data-block]` ancestor to prevent striped highlighting in nested structures. Dead code (`getFullBlockSelectionRects`, `MultiBlockResult.rects`) was removed.

## Contribution

Opened by first-time contributor @giteshsarvaiya to resolve #79643. The PR underwent iterative review with @alecgeatches and @ingeniumed, who identified scroll-to-avatar regressions and redundant type definitions. The author refined the implementation by extracting shared DOM traversal logic into `getOrderedBlockRange`, switching `usersWithAvatar` to key by user ID, and simplifying the discriminated union. Post-implementation AI-assisted review surfaced four additional correctness and null-safety bugs, which were addressed in follow-up commits before merge. The approach shifted from a uniform CSS outline to a two-track overlay/border system after reviewers noted that borders alone failed to convey selection semantics for text content.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
