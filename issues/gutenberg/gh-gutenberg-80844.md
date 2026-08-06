# #80844: Table of Contents: Replace "Convert to static list" with confirmed "Detach" action

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Table of contents (experimental)`
- **Merged:** [`36793e1`](https://github.com/WordPress/gutenberg/commit/36793e11e9476e312a34c69365c2a8dbf8553e43)
- **Discussion:** [#80844](https://github.com/WordPress/gutenberg/pull/80844) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Table of Contents block’s toolbar button is renamed from “Convert to static list” to “Detach” and now requires confirmation via a dialog before executing. The change also replaces the deprecated v1 `values` attribute serialization with proper `core/list` and `core/list-item` blocks, preventing an automatic v1-to-v2 migration trap on every detach.

## Impact

- **Editors & site owners:** The detach action now shows a confirmation dialog explaining that manual editing will be enabled but automatic heading updates will stop. Terminology aligns with the Gallery block’s detach convention.
- **Plugin & theme developers:** No direct API changes, but the block’s serialized output now uses the v2 list schema (`core/list` + `core/list-item`) instead of the deprecated `values` attribute. This eliminates forced client-side migrations on detached instances.
- **No action required** for existing sites; the change applies to new detach operations.

## Technical details

In `packages/block-library/src/table-of-contents/edit.js`, the toolbar logic is extracted into a `TableOfContentsToolbar` component. The detach action now wraps the block replacement in a `__experimentalConfirmDialog` and delegates serialization to a new `createListItemBlocks` utility in `utils.ts`. This utility recursively builds `core/list-item` blocks with properly escaped `content` and nested `core/list` blocks, using `escapeAttribute` and `escapeEditableHTML` instead of the previous `renderToString` approach that populated the deprecated `values` attribute.

Before:
```js
replaceBlocks(
  clientId,
  createBlock( 'core/list', {
    ordered,
    values: renderToString( <TableOfContentsList nestedHeadingList={ headingTree } ordered={ ordered } /> ),
  } )
)
```

After:
```js
replaceBlocks(
  clientId,
  createBlock( 'core/list', { ordered }, createListItemBlocks( headingTree, ordered ) )
)
```

## Contribution

Opened by @Mamaduka and merged after review from @ramonjd, @jasmussen, @andrewserong, and @fabiankaegy. The PR addresses prior discussion in #80613, standardizing the “Detach” terminology and fixing the serialization trap. Reviewers noted minor base-style mismatches between the detached list and the TOC block, which were acknowledged but not blocked from shipping.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
