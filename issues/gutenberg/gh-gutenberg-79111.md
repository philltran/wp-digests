# #79111: Icons block: insert an icon by default

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Icon`
- **Merged:** [`6f218ce`](https://github.com/WordPress/gutenberg/commit/6f218ce31b31342f367a8932afa27090635ba0b7)
- **Discussion:** [#79111](https://github.com/WordPress/gutenberg/pull/79111) · 11 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Icon block now inserts with a default `core/info` icon instead of displaying a gray placeholder. This change removes the empty placeholder state for newly inserted blocks, giving editors immediate visual feedback and clarifying the block’s purpose. Existing posts and templates remain unaffected, as the logic only triggers on fresh block insertion.

## Impact

- **Block & theme developers:** No code changes required. The block’s `example.attributes.icon` now reflects `core/info`, which may slightly alter block preview thumbnails in the inserter.
- **Site owners & editors:** Newly inserted Icon blocks will render an info icon immediately, removing the previous gray placeholder. No migration or configuration is needed.
- **Headless & REST consumers:** No impact; the block’s saved attributes and REST schema remain unchanged.

## Technical details

The diff modifies `packages/block-library/src/icon/edit.js` to detect fresh block insertion using `blockEditorStore.wasBlockJustInserted( clientId )`. When a block is newly inserted and lacks an `icon` attribute, a `useEffect` hook calls `setAttributes( { icon: 'core/info' } )` while suppressing undo history via `__unstableMarkNextChangeAsNotPersistent()`. This ensures existing content loaded from the database is never silently altered. The `Edit` component signature now accepts a `clientId` prop to support the insertion detection. The block registration in `packages/block-library/src/icon/index.js` also updates the `example.attributes.icon` value from `core/audio` to `core/info`.

## Contribution

Opened by @jasmussen and merged with co-authors @t-hamano and @fcoveram, the PR emerged from a UX review of the Icon block’s empty placeholder. @fcoveram raised concerns about placeholder consistency across blocks like Image, suggesting a more systematic approach. @jasmussen pushed back, arguing that a default icon improves immediate usability and template design, and that Image blocks cannot logically share a default. The discussion settled on a targeted fix that only applies to newly inserted blocks, preserving backward compatibility. A secondary randomization approach mentioned in the PR description was not included in the final diff.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
