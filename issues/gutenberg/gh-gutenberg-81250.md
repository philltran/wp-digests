# #81250: Block editor: unify what a new sibling block inherits

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`1e267a5`](https://github.com/WordPress/gutenberg/commit/1e267a50fab29f1b005631009f00d8c272423329)
- **Discussion:** [#81250](https://github.com/WordPress/gutenberg/pull/81250) · 2 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Creating a new block next to an adjacent block of the same type now inherits attributes by one rule on every path: the block appender, the inserter, insert before/after, and Enter at the edge of the text. Everything is copied except attributes with the `content` role and the `metadata` attribute. The hand-maintained `attributesToCopy` list on direct-insert blocks (used by `core/buttons`) is removed. The practical fix: pressing Enter at the end of a styled Button's text now yields a matching, empty Button instead of an unstyled one.

## Impact

**Plugin & theme developers**
- `attributesToCopy` is no longer read from the object returned by the `getDirectInsertBlock` selector, and it is removed from the `WPDirectInsertBlock` typedef and the generated docs. Code that supplied it via a container block's default/direct-insert block (the pattern `core/buttons` used) no longer controls what gets copied. Copying is now derived from attribute roles.
- Custom blocks that serve as a direct-insert block and want a styled sibling to carry over should mark text-like attributes with `role: 'content'`. Any attribute without that role is now copied, including ones not previously in an allowlist.
- The `anchor` attribute is copied (as with Duplicate and copy/paste), so duplicate anchors are possible; upstream defers this to the wider editor-wide question in #32970.

**Site owners / editors**
- Enter at the end of a Button's text now produces a Button with the same styling and empty text. Mid-text Enter behavior is unchanged, as is the Buttons appender for the previously allowlisted attributes.

**Headless / REST / hosting:** no impact; this is editor-side behavior only.

## Technical details

A new helper, `getSiblingBlockAttributes( blockName, attributes )` in `packages/block-editor/src/utils/sibling-block-attributes.js`, returns the adjacent block's attributes minus the names from `getBlockAttributesNamesByRole( blockName, 'content' )`, minus `metadata`. `metadata` is excluded explicitly because it has no attribute definition that could carry a role. It returns `{}` when no attributes are passed.

Call sites in the diff:
- `components/inserter/index.js`: `insertOnlyAllowedBlock()`'s `getAdjacentBlockAttributes()` no longer takes an `attributesToCopy` list or filters by it; it returns `getSiblingBlockAttributes( blockName, adjacentAttributes )`.
- `store/actions.js`, `insertBeforeBlock` / `insertAfterBlock`: the `attributesToCopy` loop is replaced with a spread of `getSiblingBlockAttributes(...)`, guarded so it applies only when `select.getBlockName( clientId ) === directInsertBlock.name`.
- `store/actions.js`, `__unstableSplitSelection`: in the `createEmpty()` branch used when the default block is not insertable at the root, the same-type fallback block is now created with `getSiblingBlockAttributes( name, blockAttributes )`. The default-block path is unchanged.
- `store/selectors.js`: the `attributesToCopy` property is dropped from the `WPDirectInsertBlock` JSDoc; docs in `data-core-block-editor.md` are updated.
- `block-library/src/buttons/edit.js`: `DEFAULT_BLOCK` no longer lists `backgroundColor`, `border`, `className`, `fontFamily`, `fontSize`, `gradient`, `style`, `textColor`, `width`.

Mid-text splitting is untouched (both halves keep their attributes). An e2e test in `buttons.spec.js` checks that Enter at the end of a Button with `backgroundColor`, `textColor` and `anchor` set yields a second Button with the colors but the typed `text`. Note the test's expected attributes for the second button omit `anchor`, though the PR text and the helper say the anchor is copied; `toMatchObject` does not fail on extra properties.

The CHANGELOG entry was added under a duplicated `### Enhancements` heading.

## Contribution

Authored by @ellatrix and merged with only bot comments in the thread. The PR description traces the history: #37904/#37905 (2022) added `attributesToCopy` so the inserter matched Enter, and the split unification in #54543 (2023) silently stopped Enter from copying at the text edge. It is framed as prep for #81239, and the duplicate-anchor question is deliberately deferred to #32970.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
