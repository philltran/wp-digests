# #79004: Inserter: Fix error being thrown for spoken message when inserting default/direct block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Focus] Accessibility (a11y)`, `[Feature] Inserter`, `[Package] Block editor`
- **Merged:** [`36b1691`](https://github.com/WordPress/gutenberg/commit/36b1691738f709b94c89127a2deaac79c6505d2d)
- **Discussion:** [#79004](https://github.com/WordPress/gutenberg/pull/79004) · 4 comments · 0 reactions

## Summary

Fixes a TypeError thrown in the block editor inserter when inserting default or direct block types. Previously, accessing `allowedBlockType.title` failed when the block type was null/false, breaking screen reader announcements. The update safely resolves block labels using an experimental function and active variation titles, ensuring reliable accessibility feedback without crashing.

## Impact

- **Screen Reader & Accessibility Users**: Spoken messages for block insertion (e.g., "Page link block added") are now generated reliably instead of throwing errors or reading "undefined".
- **Editor Plugin/Theme Developers**: No API changes or migration required. This is an internal `@wordpress/block-editor` component fix.
- **No action required** for end users or headless consumers.

## Technical details

- **File**: `packages/block-editor/src/components/inserter/index.js`
- **Problem**: The `allowBlockInsertion` callback passed a direct/default block name, resulting in `allowedBlockType` being `null` or `false`. Accessing `.title` on these values caused a TypeError that silenced the `speak()` announcement.
- **Solution**: The PR modernizes the component's store access and safely resolves the spoken label:
  ```diff
- import { useDispatch, useRegistry, useSelect } from '@wordpress/data';
+ import { useDispatch, useSelect } from '@wordpress/data';
-
- const registry = useRegistry();
+ const { getBlock } = useSelect( blockEditorStore );
+ const { getActiveBlockVariation, getBlockType } = useSelect( blocksStore );
  ```
- **Label Resolution**: Replaces direct `.title` access with `getBlockLabel()` (imported as `__experimentalGetBlockLabel` from `@wordpress/blocks`). If the returned label matches the type's default title, it falls back to the active variation's title via `getActiveBlockVariation()`. The `speak()` call is now guarded by an `if ( blockLabelToInsert )` check.
- **Scope**: Affects only the Inserter component's internal selection logic and A11y feedback mechanism.

## Contribution

Opened by @talldan to resolve a crash exposed during direct/default block insertion. Co-authored by @ntsekouras and @Mamaduka, who noted the fix aligns with broader title resolution efforts (referencing #58641). The PR was refined through review to clean up legacy `useRegistry()` usage alongside the accessibility patch. Merged as commit 36b16917.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
