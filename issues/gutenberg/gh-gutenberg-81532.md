# #81532: Block editor: add an inserter to the parent selector

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Feature] Inserter`, `[Package] Block library`, `[Package] Block editor`
- **Merged:** [`dbe1c1c`](https://github.com/WordPress/gutenberg/commit/dbe1c1ccab1a056ebcc2361927a0a2043daf417b)
- **Discussion:** [#81532](https://github.com/WordPress/gutenberg/pull/81532) · 14 comments · 4 reactions
- **Usefulness:** 3/5

## Summary

The block parent selector in the editor toolbar now carries a second button — a plus that inserts a sibling block *after* the currently selected child, without first having to select the parent. Previously the only way to append a sibling from inside a nested block was to know that Enter does it, or to select the parent and hunt for its corner appender or custom toolbar inserter. `BlockParentSelector` now renders the parent button and a standard `Inserter` together inside a `ToolbarGroup`, and Column's default appender was removed as a consequence. This fixes gutenberg#47200 (the long-standing "how do I add another gallery image" complaint) and applies to every block with inner blocks, not just the ones that shipped a bespoke appender.

## Impact

**Block & plugin developers**

- Any custom block with inner blocks now gets a toolbar plus on its children automatically — no registration change, no opt-in flag. There is deliberately no per-block API to enable or disable it.
- The only opt-out is implicit: a parent block type that defines `merge` or supports `__experimentalOnMerge` (core List, Quote) suppresses the button, on the reasoning that those wrappers grow by typing.
- What the plus *does* depends on your existing config: a single insertable type inserts directly, a declared `defaultBlock` inserts directly, otherwise the quick inserter opens. If your block restricts `allowedBlocks` or declares a default block, review what the plus produces.
- The `Inserter` component still hides itself when nothing is insertable, so Page List and content-locked templates are unaffected.

**Anyone with editor E2E tests**

- The block toolbar gained a focusable stop between the parent selector and the block-type switcher. Roving-tabindex/arrow-key tests need an extra `ArrowRight`; core's `toolbar-roving-tabindex.spec.js` was updated for exactly this.
- Selector churn: the appender inside a non-empty Column block is gone, so canvas-based `Add Block` locators must move to the toolbar's `Add block` button (see the `block-template-lock.spec.js` change).

**Site owners / editors**

- Behavior change only, no action required. Selecting a Column and pressing the toolbar plus now adds another *Column*, not a block inside it — the in-column appender that used to do the latter has been removed.

**Translators / string consumers**

- Two new strings with contexts: `_x( 'Add %s', 'directly add the only allowed block' )` and `_x( 'Add block', 'Generic label for block inserter button' )`.

## Technical details

The work is in `packages/block-editor/src/components/block-parent-selector/index.js`. The `useSelect` mapping was extended beyond `parentClientId` to also return `nextSiblingClientId` (via `getNextBlockClientId`) and a `showInserter` boolean:

```js
const parentBlockType = getBlockType( getBlockName( _parentClientId ) );
const isTextFlowWrapper =
	parentBlockType?.merge ||
	hasBlockSupport( parentBlockType, '__experimentalOnMerge' );
showInserter:
	!! _parentClientId &&
	_parentClientId === immediateParentClientId &&
	! isTextFlowWrapper,
```

The `_parentClientId === immediateParentClientId` check matters: when `getParentSectionBlock()` resolves a section further up the tree, its content is locked, so no button is rendered.

The parent `ToolbarButton` was extracted into a `parentButton` variable. When `showInserter` is false it is rendered bare, as before; when true it is wrapped with an `Inserter` in a real `ToolbarGroup` so the pair reads and sizes as one toolbar segment:

```jsx
<ToolbarGroup>
	{ parentButton }
	<Inserter
		position="bottom right"
		rootClientId={ parentClientId }
		clientId={ nextSiblingClientId }
		isAppender={ ! nextSiblingClientId }
		__experimentalIsQuick
		renderToggle={ ( { onToggle, isOpen, disabled, blockTitle, hasSingleBlockType } ) => (
			<ToolbarButton className="block-editor-block-parent-selector__inserter" … icon={ plus } />
		) }
	/>
</ToolbarGroup>
```

Targeting the sibling *after* the selection is done purely with `rootClientId` + `clientId`; when the selected block is last, `isAppender` takes over. The toggle label is `Add %s` (lowercased `blockTitle`) when `hasSingleBlockType`, otherwise `Add block`.

`packages/block-editor/src/components/block-tools/style.scss` gains `:has()`-based rules: `.has-parent:has(.block-editor-block-parent-selector__inserter)` widens the toolbar's left margin, the parent-selector container takes the border/background/radius so the group paints as one box, and `.block-editor-block-parent-selector__inserter` draws the familiar black square via `::after`. Under the `show-icon-labels` preference the icon is hidden and `::after` renders `content: attr(aria-label)` instead.

One block-library change, in `packages/block-library/src/column/edit.js`:

```diff
 renderAppender: hasChildBlocks
-	? undefined
+	? false
 	: InnerBlocks.ButtonBlockAppender,
```

`undefined` falls back to the default appender; `false` removes it, so a non-empty Column no longer shows a plus at its bottom.

Test coverage: a new `test/e2e/specs/editor/various/parent-selector-inserter.spec.js` (gallery insert-between, plus a Group-shows / List-hides assertion), a Buttons case asserting the new block lands between the two existing ones, and updates to `block-template-lock.spec.js` and `toolbar-roving-tabindex.spec.js` for the new toolbar shape.

## Contribution

Authored by @ellatrix against issue #47200. The substantive review debate came from @andrewserong, who flagged the Columns case — the toolbar plus adds a sibling Column while the in-column appender adds a child — and floated making the feature **opt-in per block** so only Gallery, Social Icons and similar used it. @ellatrix rejected that in favor of shipping it as a general rule and tweaking later, and resolved the Columns ambiguity in the other direction instead: by deleting the Column appender, citing #60307 (Quote) and Gallery as precedent, with a broader appender refactor planned as a follow-up in the same release cycle. @jasmussen signed off on trying it while noting a possible design follow-up. Props also recorded for @richtabor, @hanneslsm and @danielbachhuber.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
