# #81129: List View: Narrow the dependants of the memoized List View tree

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Feature] List View`, `[Package] Block editor`
- **Merged:** [`550ada7`](https://github.com/WordPress/gutenberg/commit/550ada718ba830ddafa511a7eaca44e466d67362)
- **Discussion:** [#81129](https://github.com/WordPress/gutenberg/pull/81129) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This PR optimizes the block editor’s List View by narrowing the memoization dependencies of `getListViewClientIdsTree`. Previously, the selector tracked `state.blocks.attributes` unconditionally, causing the entire List View tree to rebuild on every keystroke because the attributes reducer returns a new `Map` on each `UPDATE_BLOCK_ATTRIBUTES` action. By conditionally including those state slices only when `state.editedContentOnlySection` is active, the memoized tree now remains stable during normal typing, significantly improving List View performance.

## Impact

- **Plugin & theme developers:** No action required. The change modifies internal memoization logic in `@wordpress/block-editor` and does not alter public selectors, hooks, or block.json schemas.
- **Hosting & platform teams:** No configuration, migration, or upgrade steps needed. The change ships as a transparent performance improvement.
- **Headless & REST consumers:** No impact. The optimization is isolated to the client-side block editor store and does not affect REST API routes or data shapes.
- **Breaking changes / deprecations:** None. The `createRegistrySelector` wrappers removed from `getEnabledClientIdsTree` and `getListViewClientIdsTree` were internal dead code and never part of the public API.

## Technical details

The diff modifies `packages/block-editor/src/store/private-selectors.js`. It removes the now-unnecessary `createRegistrySelector` wrapper from both `getEnabledClientIdsTree` and `getListViewClientIdsTree`, as neither reads another store and the editor mode value was moved to `derivedBlockEditingModes` in #67026. More importantly, it restructures the dependency array for `getListViewClientIdsTree`:

```javascript
// Before
createSelector( getListViewClientIdsTreeUnmemoized, ( state ) => [
    state.blocks.order,
    state.derivedBlockEditingModes,
    state.blocks.blockEditingModes,
    state.blocks.parents,
    state.blocks.byClientId,
    state.blocks.attributes,
    state.blockListSettings,
    state.editedContentOnlySection,
    state.settings,
] )

// After
createSelector(
    getListViewClientIdsTreeUnmemoized,
    ( state ) => [
        state.blocks.order,
        state.derivedBlockEditingModes,
        state.blocks.blockEditingModes,
        state.blocks.parents,
        state.editedContentOnlySection,
        ...( state.editedContentOnlySection
            ? [ state.blocks.byClientId, state.blocks.attributes, state.blockListSettings, state.settings ]
            : [] ),
    ]
)
```

The `UPDATE_BLOCK_ATTRIBUTES` action replaces `state.blocks.attributes` with a new `Map` instance, which previously broke the shallow equality check in `useListViewClientIds`. The conditional dependency array ensures the tree only rebuilds when actually editing a content-only section, while remaining referentially stable during standard typing. Test coverage in `packages/block-editor/src/store/test/private-selectors.js` verifies referential stability across `UPDATE_BLOCK_ATTRIBUTES` and correct tree reconstruction when `editedContentOnlySection` changes.

## Contribution

Opened by @Mamaduka as a follow-up to #73997, this PR addresses a performance regression where typing triggered full List View tree rebuilds. The author identified that four state dependencies were only required inside the `editedContentOnlySection` branch and removed legacy `createRegistrySelector` wrappers that had become dead code after #67026. The change was reviewed and merged with minimal discussion, focusing on the memoization optimization and corresponding test updates.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
