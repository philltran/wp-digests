# #78483: RTC: Fix every update block refresh when a peer edits with in the code editor

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Task`, `[Package] Core data`, `[Feature] Real-time Collaboration`
- **Merged:** [`ea0d91b`](https://github.com/WordPress/gutenberg/commit/ea0d91be40e283219bd9056bd1cbe55dc116d724)
- **Discussion:** [#78483](https://github.com/WordPress/gutenberg/pull/78483) · 6 comments · 0 reactions

## Summary

Fixes a real-time collaboration (RTC) bug where every block on a collaborator's screen would unmount and remount — causing embed iframes (YouTube, PDF, etc.) to flash and reload — on each keystroke while the other user typed in the Code Editor. The root cause was in the CRDT sync layer: the `PostTextEditor` component dispatches post updates with `{ blocks: undefined, content: '...' }` on every keystroke, and `applyPostChangesToCRDTDoc` interpreted the absent `blocks` property as a signal to wipe and rebuild the entire `Y.Array`, generating fresh `clientId`s for every block. New `clientId`s forced every block to remount. Two targeted fixes in `packages/core-data/src/utils/crdt.ts` prevent both the full-array replacement and the clientId overwrite.

## Impact

**Plugin & theme developers** — No action required; no public APIs, hooks, filters, or block.json fields changed.

**Platform & hosting teams running WordPress with RTC enabled** — No configuration changes needed. The fix is self-contained in the `@wordpress/core-data` package's internal CRDT utilities.

**Editors using Real-Time Collaboration** — Embed blocks (YouTube, PDF, and all other embed variants) and all other blocks will no longer flash or reinitialise their iframes when a collaborator edits in the Code Editor. Paragraph text still propagates live across sessions.

## Technical details

All changes are in `packages/core-data/src/utils/crdt.ts`.

**1. `applyPostChangesToCRDTDoc` — new `mergeContentWithoutBlocks()` path**

Previously, when the incoming update had `blocks === undefined` (the exact shape dispatched by `packages/editor/src/components/post-text-editor/index.js` on each keystroke), the function fell into a branch that deleted the entire `Y.Array` and re-inserted freshly-parsed blocks, assigning brand-new `clientId`s to every block in the document.

After the fix, when `blocks` is absent but `content` is present, a new helper `mergeContentWithoutBlocks()` parses the content string and routes the resulting block list through the existing `mergeCrdtBlocks()` diff, leaving the `Y.Array` entries in place and preserving their identity.

```ts
// Before — content-only update wiped the Y.Array
if ( ! blocks ) {
  const parsed = parse( content );
  yBlocks.delete( 0, yBlocks.length );
  yBlocks.insert( 0, parsed ); // fresh clientIds → every block remounts
}

// After — content-only update goes through the diff
if ( ! blocks && content ) {
  mergeContentWithoutBlocks( yDoc, content ); // calls mergeCrdtBlocks internally
}
```

**2. `mergeCrdtBlocks()` update loop — new `case 'clientId'` skip**

Even with the above fix, the block actively being edited in the Code Editor would still receive a freshly-parsed `clientId` on the next sync tick, causing that specific block to remount. A new `case 'clientId':` branch in the per-property update loop now skips the `clientId` key entirely, so the local Y.Doc's stable `clientId` is never overwritten by an incoming parsed value.

The only build artifact that changed is `build/scripts/core-data/index.min.js` (+64 B, 0.2%).

## Contribution

PR #78483 was opened and merged by @alecgeatches (commit `ea0d91b`), closing issue #76598. Props were recorded for @maxschmeling, @chriszarate, and @desrosj. The PR discloses that Claude Code was used for bug investigation, fix planning, CRDT-layer implementation, and unit test authorship. The discussion contained 6 comments; no alternative approaches are documented in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
