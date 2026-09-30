# #82475: Fix auto-capitalization after Enter on iOS Safari

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Feature] Writing Flow`, `[Package] Editor`, `[Package] Block library`, `[Package] Rich text`, `[Package] Block editor`
- **Merged:** [`cf5aae5`](https://github.com/WordPress/gutenberg/commit/cf5aae5bc8b416f6c96025219b7e2913981dc0bb)
- **Discussion:** [#82475](https://github.com/WordPress/gutenberg/pull/82475) · 4 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Enter handling in the Gutenberg editor now runs on the native `beforeinput` event (`insertParagraph` / `insertLineBreak`) instead of `keydown`. On iOS Safari, moving focus to the new block while the keydown was still being handled left the keyboard's auto-capitalization state stale, so the first word in a new paragraph wasn't capitalized. The PR also fixes caret placement in empty fields so the caret sits before the zero-width placeholder character, which the iOS keyboard otherwise reads as a preceding character.

## Impact

- **Site owners / editors on iOS Safari:** a new paragraph created with Enter (including from the post title and in lists) now starts with shift on and the next word capitalized.
- **Plugin & theme developers:** no API removals or new public hooks. Custom blocks or plugins that listen for `keydown` Enter on editables to intercept or cancel block splitting may see different ordering. Core's Enter handling for paragraph, list item and post title no longer happens on `keydown`, so `keydown` handlers no longer see `defaultPrevented` set by core for Enter in those cases. Code that relied on that, or on calling `preventDefault()` on `keydown` Enter to stop splitting, should be retested. The deprecated `onSplit` + `onReplace` path is still flagged, now on the `beforeinput` event (`event.__deprecatedOnSplit`).
- **Desktop:** intended to behave as before for Enter, Shift+Enter, and Enter in lists and quotes.
- **Hosting / headless:** not affected.

## Technical details

**Enter moves from `keydown` to `beforeinput`:**
- `block-editor/.../rich-text/event-listeners/enter.js`: the window-level default `keydown` listener (which always `preventDefault()`ed Enter) is replaced by `onDefaultBeforeInput`, which handles `insertParagraph` and `insertLineBreak`. A new capture-phase `beforeinput` listener on the element sets `event.__deprecatedOnSplit = true` when both `onReplace` and `onSplit` are present; the `keydown` branch that used to set it now just excludes that case from the line-break fallback.
- `block-editor/.../writing-flow/use-input.js`: `onBeforeInput` now calls `onKeyDown( event )` for `insertParagraph` when not default-prevented and no multi-selection, returning early if it was cancelled. The `onKeyDown` Enter check accepts `event.inputType === 'insertParagraph'` as well as `keyCode === ENTER`, and skips `keydown` when the active element is `isContentEditable` (editables are handled on `beforeinput`).
- `block-library/src/paragraph/use-enter.js` and `list-item/hooks/use-enter.js`: the capture-phase `keydown` listeners via `subscribeOwnedListener` become `beforeinput` listeners checking `inputType === 'insertParagraph'`; the `ENTER` keycode imports are removed.
- `editor/.../post-title/index.jsx`: the React `onKeyDown` is replaced by a native `beforeinput` listener attached through `useRefEffect` and merged into the `h1` ref (React's `onBeforeInput` is synthesized and lacks `inputType`). It calls `preventDefault()` and `onEnterPress()` for `insertParagraph`/`insertLineBreak`.

**Caret placement (`rich-text/.../input-and-selection.js`):**
- Removes `fixPlaceholderSelection()`.
- In `handleSelectionChange`, when `text.length === 0` it now calls `applyRecord( { ...oldRecord, start, end } )` immediately, before the `selectionSnapshot` is taken, so the resulting selection-change event is seen as already processed. The code comment notes re-applying from a later task moves the caret on iOS and loses the capital.
- The focus handler for nested editables now also does `window.queueMicrotask( handleSelectionChange )` so the caret placed after focus is synchronized in the same task.

**Test:** a new e2e spec in `splitting-merging.spec.js` asserts the event order `keydown`, `beforeinput:insertParagraph`, `focusin`, that the `beforeinput` is cancelled, that the caret is at offset zero of the new field and not inside `[data-rich-text-placeholder]`, and that typing works. It carries a "do not alter unless re-testing iOS auto-capitalization" comment.

Bundle size changes are small (about +67 B overall).

## Contribution

Authored by @ellatrix with the assistance of Claude Code, closing the long-standing issue #63261. The diagnosis in the description found two separate causes: the caret position relative to the placeholder character, and the `keydown`-based prevent not resetting Safari's internal state. The automated CodeRabbit review raised no actionable comments, and the record shows no further design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
