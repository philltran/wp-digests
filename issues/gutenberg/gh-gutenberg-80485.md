# #80485: Core Data: Move EntitiesSavedState component out of editor (rebase of #71948)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @enejb
- **Labels:** `[Type] Performance`, `[Package] Core data`, `[Package] Editor`, `[Package] Edit Site`
- **Merged:** [`164a505`](https://github.com/WordPress/gutenberg/commit/164a505916f755b245620ca68f67cb53a0daf614)
- **Discussion:** [#80485](https://github.com/WordPress/gutenberg/pull/80485) · 1 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `EntitiesSavedStates` component, its hooks, and the `saveDirtyEntities` action have been moved from `@wordpress/editor` to `@wordpress/core-data`. This architectural shift centralizes entity save-state management within the core data layer, reducing cross-package coupling and aligning the component's lifecycle with the store it queries. The move also updates the `saveDirtyEntities` action to use the latest trunk implementation, adding support for custom `onSave` callbacks and `successNoticeContent` parameters.

## Impact

- **Plugin & theme developers:** No breaking changes to public APIs; `@wordpress/editor` re-exports `EntitiesSavedStates` and `useEntitiesSavedStatesIsDirty` for backward compatibility. However, any custom code importing these directly from `@wordpress/editor` should migrate to `@wordpress/core-data`'s new private exports to avoid future deprecation.
- **Editor/Block developers:** The `saveDirtyEntities` action signature now accepts an `onSave` function and `successNoticeContent` string, allowing finer control over post-save behavior and UI feedback.
- **Platform & hosting teams:** No action required. This is a JavaScript package refactor with no PHP, REST API, or database impact.

## Technical details

The diff relocates `packages/editor/src/components/entities-saved-states/` to `packages/core-data/src/components/entities-saved-states/`, along with `get-template-info.js` and `get-template-part-icon.js` utils. The `saveDirtyEntities` action is added to `packages/core-data/src/private-actions.js`, replacing `editorStore` dispatches with `STORE_NAME` (core-data) and integrating `noticesStore` for success/error feedback. The action now calls `registry.dispatch( blockEditorStore ).__unstableMarkLastChangeAsPersistent()` before saving and chains `await onSave()` after successful saves. New private APIs (`EntitiesSavedStates`, `EntitiesSavedStatesExtensible`, `getTemplateInfo`, `getTemplatePartIcon`, `useEntitiesSavedStatesIsDirty`) are registered in `packages/core-data/src/private-apis.ts` and documented in `docs/private-apis.md`. The `@wordpress/core-data` package.json gains dependencies on `@wordpress/components`, `@wordpress/global-styles-engine`, `@wordpress/icons`, `@wordpress/notices`, and `clsx`. Consumers like `edit-site/src/components/editor/use-editor-title.js` and `post-excerpt/panel.js` are updated to import `getTemplateInfo` from `core-data` instead of `editor`.

## Contribution

This PR is a rebase of @youknowriad's original #71948, which had fallen ~4,400 commits behind trunk. @enejb handled the rebase, intentionally preserving trunk's updated `saveDirtyEntities` logic (including the `await onSave()` pattern from #79850) and keeping the `__unstableMarkLastChangeAsPersistent()` call that the original PR's cleanup commit had removed. Independent trunk consumers were updated to pull template utilities from `core-data`. The PR merged with minimal review discussion, reflecting a straightforward structural migration rather than a feature debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
