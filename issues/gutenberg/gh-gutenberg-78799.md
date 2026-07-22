# #78799: Sync editor settings in layout effect (fixes autosave e2e)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jsnajdr
- **Labels:** `[Package] Editor`, `[Type] Flaky Test`
- **Merged:** [`6832cf4`](https://github.com/WordPress/gutenberg/commit/6832cf4169fe158bdc2a7dbc4e96c4de847e8e61)
- **Discussion:** [#78799](https://github.com/WordPress/gutenberg/pull/78799) · 17 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

A race condition in editor initialization that caused duplicate autosave notices and a flaky e2e test is fixed by synchronizing editor settings before the first paint. The change moves `updateEditorSettings` in `ExperimentalEditorProvider` from `useEffect` to `useLayoutEffect`, ensuring child components like `LocalAutosaveMonitor` receive accurate `getEditorSettings().autosave` values on mount. It also refactors the autosave notice logic to read settings synchronously within the effect and removes a test guard that was masking a CRDT timestamp race condition.

## Impact

- **Plugin & theme developers:** No public API changes or deprecations. Editor initialization timing is now stricter, but stable editor APIs remain unaffected.
- **E2E test authors:** The `stillHasRemoteAutosave` skip guard in `autosave.spec.js` was removed; tests expecting remote autosave notices must now ensure proper setup timing.
- **No action required** for site owners, plugin authors, or theme developers using standard editor interfaces.

## Technical details

- In `packages/editor/src/components/provider/index.js`, the `useEffect` calling `updateEditorSettings( settings )` is replaced with `useLayoutEffect`. This guarantees settings are applied before the browser paints, preventing child effects from reading default values.
- In `packages/editor/src/components/local-autosave-monitor/index.js`, `useSelect` is no longer used to read `hasRemoteAutosave` at the component top level. Instead, `useRegistry()` is used, and `registry.select( editorStore ).getEditorSettings()` is called synchronously inside the effect. `useDispatch` calls for `noticesStore` and `editorStore` are also moved inside the effect to avoid stale closures.
- In `test/e2e/specs/editor/various/autosave.spec.js`, the `stillHasRemoteAutosave` conditional skip was removed. `{ delay: 100 }` was added to `page.keyboard.type()` to space out autosave and publish timestamps, avoiding a race with `persistCRDTDoc` updates to `post_modified`.

## Contribution

Opened and merged by @jsnajdr to address a React 19 migration-induced scheduling regression. Reviewers (@Mamaduka, @alecgeatches) discussed the root cause, noting that moving settings sync out of the React tree would be ideal but too complex for this release. The CRDT timestamp race was acknowledged as a separate backend issue, with a future `wp-sync` endpoint proposed but deferred. The fix landed as a targeted layout effect shift and test stabilization.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
