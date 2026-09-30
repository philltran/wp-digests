# #80070: Block Editor: Fix style / nested attribute overwrite during multi-selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Intenzi
- **Labels:** `[Type] Bug`, `[Feature] Blocks`, `[Package] Block editor`, `First-time Contributor`
- **Merged:** [`cae85f3`](https://github.com/WordPress/gutenberg/commit/cae85f31efa0d25dee90ee4e80c4fd18579764cc)
- **Discussion:** [#80070](https://github.com/WordPress/gutenberg/pull/80070) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Fixes a data-overwrite bug in the Block Editor where changing a design setting (e.g. text color) on a multi-block selection replaced each block's entire `style` object with the primary block's, silently destroying distinct per-block values like background colors, borders, and margins. The fix introduces recursive diffing so only the actually-changed leaf values are propagated to every selected block. The same shallow-merge problem affected all nested object attributes, not just `style`.

## Impact

- **Site owners / editors:** No action required. Multi-block style edits now preserve each block's unrelated style values instead of overwriting them. Previously, changing text color on a 3-block selection could silently replace each block's background color with the primary block's.
- **Plugin & theme developers:** No code changes needed. If your block registers nested object attributes (`style`, `metadata`, custom config objects), multi-selection edits will now deep-merge rather than shallow-replace. No new public API is exposed; the diffing utilities are internal to `@wordpress/block-editor`.
- **Hosting & platform:** No configuration or migration changes. The fix is entirely in the editor's client-side `setAttributes` interceptor.
- **Known limitation:** When `cleanEmptyObject` collapses a partial removal (e.g. `{ border: { top: undefined } }` → `{ border: undefined }`), the editor cannot distinguish "clear all border" from "clear only top border." The implementation picks the least-destructive interpretation (preserves other values). A follow-up to revisit `cleanEmptyObject` usage at `setAttributes` call sites is anticipated.

## Technical details

New module `packages/block-editor/src/utils/multi-selection-attributes.ts` exports two functions:

- `getAttributeChanges(attributes, attributeUpdates)` — computes a recursive diff between the block's current attributes and the incoming update, restricted to keys present in the update. Returns only the changed leaf values (or `undefined` if nothing changed). Handles object removal by expanding it into per-key `undefined` entries.
- `applyAttributeChanges(attributes, changes)` — deep-merges the diff into a target block's attributes, preserving keys the diff does not mention. Prunes branches that become empty objects.

Internal helpers `diffValues` and `applyValueChange` perform the recursion. `isPlainObject` was extracted into `packages/block-editor/src/utils/object.ts` and is now shared by `hooks/style.jsx` and `hooks/utils.jsx` (the local copy in `style.jsx` was removed).

The `setAttributes` interceptor in `packages/block-editor/src/components/block-list/block.jsx` was refactored:

```js
// Before (simplified):
const clientIds = multiSelectedBlockClientIds.length
  ? multiSelectedBlockClientIds
  : [ clientId ];
updateBlockAttributes( clientIds, newAttributes );

// After:
if ( ! multiSelectedBlockClientIds.length ) {
  updateBlockAttributes( clientId, newAttributes );
  return;
}
const changes = getAttributeChanges( attributes ?? {}, newAttributes ?? {} );
if ( ! changes ) return;
const updatesByClientId = {};
for ( const selectedClientId of multiSelectedBlockClientIds ) {
  updatesByClientId[ selectedClientId ] = applyAttributeChanges(
    getBlockAttributes( selectedClientId ) ?? {},
    changes
  );
}
updateBlockAttributes(
  multiSelectedBlockClientIds,
  updatesByClientId,
  { uniqueByBlock: true }
);
```

The `{ uniqueByBlock: true }` option on `updateBlockAttributes` ensures each block receives its own computed update rather than a single shared object. Unit tests cover both exported functions in `packages/block-editor/src/utils/test/multi-selection-attributes.js`; an E2E spec in `test/e2e/specs/editor/various/multi-block-selection.spec.js` asserts background preservation during multi-selection text-color changes.

## Contribution

Opened by first-time contributor @Intenzi, who used AI assistance (Antigravity IDE / Gemini 3.5 Flash) for file identification and initial implementation. @talldan reviewed, asked permission to rebase and push additional commits, and expanded the fix to handle root-level attribute unsetting, empty-object pruning, and reference preservation to avoid unnecessary re-renders. Talldan also documented the remaining `cleanEmptyObject` ambiguity as a known limitation requiring follow-up work. The PR was merged as `cae85f3` with co-authorship credited to talldan, tellthemachines, ramonjd, ndiego, bph, annezazu, dpmehta, juanmaguitar, and t-hamano.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
