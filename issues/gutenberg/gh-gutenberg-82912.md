# #82912: Block Toolbar: Show the parent selector for blocks inside patterns

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Patterns`
- **Merged:** [`92dc78b`](https://github.com/WordPress/gutenberg/commit/92dc78bae34d022a57af8d9cba31e512b928bf81)
- **Discussion:** [#82912](https://github.com/WordPress/gutenberg/pull/82912) · 7 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The block toolbar's parent block selector was hidden for every block in `contentOnly` editing mode, a leftover check from the now-removed Write mode. This PR restores the selector for blocks inside patterns (synced with overrides, unsynced) and `contentOnly` locked blocks, and changes both the toolbar and the selector component to resolve the parent via `getEnabledBlockParents`, skipping disabled blocks so the target matches what List View and the breadcrumb display.

## Impact

- **Content editors / site owners:** The parent block button now appears in the toolbar when a block inside a synced pattern (with overrides), an unsynced pattern, or a `contentOnly` locked group is selected. Clicking it selects the nearest enabled parent (the pattern or the locked group), giving a quick way back out of nested content. No configuration or code changes required.
- **Plugin & theme developers:** No API, hook, or schema changes. The `showInserter` flag on the parent selector now also suppresses the "+" button when the resolved parent is a section, so no new inserter appears in locked pattern content. No action required.
- **Headless / REST consumers:** No effect.

## Technical details

Two files in `packages/block-editor/src/components/` are modified:

**`block-toolbar/index.jsx`** — The `showParentSelector` condition changes from:

```js
editingMode !== 'contentOnly' &&
```
to:
```js
( editingMode !== 'contentOnly' || !! parentSection ) &&
```
so the selector is shown when the block sits inside a section (pattern) even though its editing mode is `contentOnly`. Parent resolution switches from `parentSection ?? parents[ parents.length - 1 ]` to `getEnabledBlockParents( selectedBlockClientId, true )[ 0 ]`, which walks up the tree skipping blocks whose editing mode is `disabled`.

**`block-parent-selector/index.jsx`** — The same `getEnabledBlockParents` call replaces the old `parentSection ?? immediateParentClientId` logic. The `showInserter` condition gains an additional `! parentSection` guard alongside the existing `_parentClientId === immediateParentClientId` and `! isTextFlowWrapper` checks, preventing a "+" button from appearing when the resolved parent is a section.

E2E tests are added in `test/e2e/specs/editor/various/content-only-lock.spec.js` (verifying a button inside a nested disabled group selects the Buttons block, not the outer Group, and that no inserter appears) and `test/e2e/specs/editor/various/pattern-overrides.spec.js` (verifying a paragraph with overrides inside a synced pattern selects the pattern block).

## Contribution

Opened by @ramonjd. During review, @talldan spotted a bug in the demo video: a button inside a `contentOnly` locked group was selecting the outer Group instead of the intermediate Buttons block. @ramonjd confirmed the issue and switched the parent resolution to `getEnabledBlockParents`, which skips disabled blocks, then updated the video. @fcoveram reviewed the recording and approved. The PR was merged with co-author credits to ramonjd, talldan, fcoveram, youknowriad, jasmussen, scrobbleme, and artemiomorales.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
