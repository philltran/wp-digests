# #82911: Keep the iOS keyboard capitalized after Return inside the editing host

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Block editor`
- **Merged:** [`cced808`](https://github.com/WordPress/gutenberg/commit/cced8086ad63428df283dddb6dbd1ef5932e2c53)
- **Discussion:** [#82911](https://github.com/WordPress/gutenberg/pull/82911) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes an iOS bug where pressing Return inside the editing host caused the keyboard to drop to lowercase and the next Return to insert a line break instead of a new paragraph. The root cause was twofold: the newly inserted block's field mounted without a caret, triggering a focus bounce that reset iOS capitalization, and the rich-text Enter listener read the iOS keyboard's always-on shift flag as Shift+Enter. The fix moves Enter handling entirely to `beforeinput` (using `inputType` to distinguish paragraph from line break) and pre-sets the selection for `editableRoot` blocks so their field mounts with a caret already placed.

## Impact

- **iOS block-editor users (editing host):** Return now keeps the keyboard capitalized and consistently creates a new paragraph. No action required.
- **Blocks opting into `editableRoot` (private API):** The store now dispatches `selectionChange` to offset 0 in the same batch as insertion or split. If a block's first-focus code previously relied on being the one to set the initial caret, it will find the selection already applied. Blocks that do *not* opt in are unaffected.
- **Plugin & theme developers (general):** No public API change. The Enter listener in `@wordpress/block-editor`'s rich-text component no longer subscribes to `keydown`; it handles everything on `beforeinput`. Custom code that was listening to or relying on the old `keydown`-based Enter path in that component should verify behavior, though the observable result (Enter = paragraph, Shift+Enter = line break) is unchanged on desktop.
- **No configuration, migration, or code changes required for the vast majority of sites.**

## Technical details

Three files changed:

**`packages/block-editor/src/components/rich-text/event-listeners/enter.js`**

The `onKeyDown` handler (which checked `event.keyCode !== ENTER` and `event.shiftKey`) is removed. The existing `onBeforeInput` handler is expanded to handle both `insertParagraph` and `insertLineBreak`:

```js
// Before: two separate handlers
function onKeyDown( event ) {
  if ( event.keyCode !== ENTER ) return;
  // …
  if ( event.shiftKey ) { /* line break */ }
  else { /* paragraph */ }
}
function onBeforeInput( event ) {
  if ( event.inputType !== 'insertParagraph' ) return;
  if ( onReplace && onSplit ) event.__deprecatedOnSplit = true;
}

// After: single handler
function onBeforeInput( event ) {
  const { inputType } = event;
  if ( inputType !== 'insertParagraph' && inputType !== 'insertLineBreak' ) return;
  // …
  if ( inputType === 'insertParagraph' && onReplace && onSplit ) {
    event.__deprecatedOnSplit = true;
  }
  if ( inputType === 'insertLineBreak' ) { /* line break */ }
  else { /* paragraph */ }
}
```

The `ENTER` import from `@wordpress/keycodes` is removed. The `keydown` subscription (`subscribeOwnedListener(element, 'keydown', onKeyDown, true)`) is removed; only the `beforeinput` subscription remains.

**`packages/block-editor/src/store/actions.js`**

`editableRootKey` is imported via `unlock( blocksPrivateApis )` from `@wordpress/blocks`.

In `insertBlocks`, after the `registry.batch` that performs the insertion, a new block checks whether the selected block's type has `editableRootKey` set. If so, it calls `findRichTextAttributeKey(blockType)` and dispatches `selectionChange(clientId, attributeKey, 0, 0)` in the same batch:

```js
if ( updateSelection && initialPosition === 0 ) {
  const clientId = select.getSelectedBlockClientId();
  const blockType = clientId && getBlockType( select.getBlockName( clientId ) );
  const attributeKey = blockType?.[ editableRootKey ] && findRichTextAttributeKey( blockType );
  if ( attributeKey ) {
    dispatch.selectionChange( clientId, attributeKey, 0, 0 );
  }
}
```

In `__unstableSplitSelection`, the empty-blocks branch (where `blocks.length` is 0) is wrapped in `registry.batch()`. After `replaceBlocks`, the tail block's type is checked for `editableRootKey` and `selectionChange(tail.clientId, tailKey, 0, 0)` is dispatched in the same batch.

**`packages/block-editor/src/store/test/actions.jsdom.test.js`**

New test case `'selects the start of an inserted editable root block text'` registers a `core/test-host` block with `[editableRootKey]: true` and a `content` rich-text attribute, calls `insertBlocks`, and asserts `dispatch.selectionChange` was called with `(block.clientId, 'content', 0, 0)`. A second assertion verifies that a block *without* `editableRootKey` does not trigger `selectionChange`.

## Contribution

Opened by @ellatrix, with co-authorship from @sarthaknagoshe2002 and @dcalhoun. The PR description notes the investigation was done with Claude Code and reviewed by the author. It references the iOS simulator harness introduced in #82565 and fixes long-standing issue #63261. The CodeRabbit automated review found no actionable comments. No design debate or alternative approaches are visible in the five discussion comments, which are predominantly bot-generated metadata.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
