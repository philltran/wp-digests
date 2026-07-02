# #79115: Blocks: Add innerContent support for static inner blocks, adopt it in the HTML block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @youknowriad
- **Labels:** `[Feature] Block API`, `[Package] Core data`, `[Package] Blocks`, `[Package] Block library`, `[Package] Block editor`, `Needs Dev Note`, `[Type] Feature`
- **Merged:** [`50ddbc1`](https://github.com/WordPress/gutenberg/commit/50ddbc1f6ed8ef8744305323d806a630715697ef)
- **Discussion:** [#79115](https://github.com/WordPress/gutenberg/pull/79115) · 14 comments · 4 reactions
- **Usefulness:** 4/5

## Summary

Introduces a new `innerContent` block support that allows blocks to retain static HTML fragments interleaved with editable inner blocks as their canonical markup source. The Custom HTML block (`core/html`) unconditionally adopts this pattern, shifting from a single raw `content` attribute to an `innerContent` array and a dedicated editor renderer. This resolves a long-standing limitation where complex hand-written HTML structures forced editors to choose between fully static or fully editable inner blocks.

## Impact

- **Block authors & plugin developers:** Can now implement the `innerContent` support to interleave static markup with editable children. Serialization/deserialization logic must account for the new fourth argument in `createBlock()`.
- **Developers modifying HTML block transforms or conversions:** Existing conversion flows (e.g., freeform migration, classic editor pasting) now pass an `innerContent` array instead of a raw `content` string to `core/html`.
- **Editors & site owners:** The `core/html` canvas preview no longer uses the `SandBox` iframe; inline scripts will not execute in the visual canvas (styles still apply). No frontend migration is required as the serialized output remains byte-identical.

## Technical details

Adds `packages/block-editor/src/components/inner-content/index.js`, a renderer that sanitizes static fragments via `safeHTML` and portals each inner block into `<wp-inner-block-slot>` placeholders. Modifies `@wordpress/blocks` data functions to accept an optional fourth `innerContent` argument in `createBlock()` and `getBlockFromExample()`, enabling the serializer to interleave static fragments with serialized children without calling `save`. Structural locks are enforced in `packages/block-editor/src/store/selectors.js` via a new `isInnerContentRoot()` helper that restricts `canMoveBlock`, `canRemoveBlock`, and `canInsertBlockType` for direct children of `core/html`. The `core/html` block's metadata (`block.json`) changes the `content` attribute from `source: "raw"`, `role: "content"` to `role: "local"` and enables `listView`. The `edit.js` component replaces the `SandBox` iframe preview with the `InnerContent` renderer, parsing block delimiters into editable inner blocks while keeping their positions fixed.

## Contribution

Opened and merged by @youknowriad with co-authors including @ellatrix, @mcsf, @mtias, @tyxla, @priethor, and @fabiankaegy. The design addresses a gap in the block grammar where interleaved HTML was parsed but inaccessible to registered blocks. Reviewers noted it aligns with locked/content-only patterns while offering a flexible alternative to top-down tree constraints. Merged just before the 7.1 release deadline.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
