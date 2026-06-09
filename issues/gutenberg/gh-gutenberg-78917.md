# #78917: Revert "wp-build: Replace getter-based exports with data properties"

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @gigitux
- **Labels:** `[Type] Bug`
- **Merged:** [`fcbe578`](https://github.com/WordPress/gutenberg/commit/fcbe57865379bcd5969119e1d9941a38f1778d0f)
- **Discussion:** [#78917](https://github.com/WordPress/gutenberg/pull/78917) · 4 comments · 0 reactions

## Summary

This merge reverts a change in the `@wordpress/build` bundler that replaced getter-based module exports with data property assignments. The previous update introduced runtime regressions tracked in issue #78697, breaking expected interop behavior for core utilities. Reverting restores the original getter-based export pattern and resolves the reported breakages while the team prepares a follow-up patch release.

## Impact

- **Plugin & theme developers**: No code changes required. The revert restores the previous module export structure, ensuring runtime utilities behave as expected in bundled environments.
- **Site owners & editors**: No action required. Editor functionality and script loading remain stable after applying this update.
- **Note**: Any custom tooling or build caches relying on the data-property export pattern introduced in #78303 will need to be refreshed to align with the restored getter behavior.

## Technical details

The diff reverts modifications in `packages/wp-build/lib/build.mjs`, rolling back the bundler transformation that switched exports from getters to direct data properties. 

**Before/after pattern (conceptual, based on restored behavior):**
```javascript
// Reverted state (original): Export via getter for proper ES interop
Object.defineProperty(exports, 'isShallowEqual', {
  get: function() { return _shallow.equal; },
  enumerable: true,
  configurable: true
});

// Previously introduced state (data property) - now reverted
exports.isShallowEqual = _shallow.equal;
```

The unified diff shows this change produces minor, consistent filesize reductions (typically -28 to -48 bytes) across `build/scripts/*.min.js` bundles. The restoration is validated by the console check `window.wp.isShallowEqual.__esModule` returning a defined value instead of `undefined`, confirming that ES module interop metadata is preserved correctly at runtime.

## Contribution

Opened and merged by @gigitux, with co-authorship from @aduth to manage the revert commit. The PR was created as a direct follow-up to #78303 after regression reports surfaced in issue #78697 and WordPress.org support channels. The core team decided to roll back the export strategy immediately to stabilize the release, noting that an alternative, safer approach would be explored once the patch release ships.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
