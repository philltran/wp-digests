# #83041: Rich text: fix mobile selection hair pin grab at edge of block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Package] Rich text`
- **Merged:** [`d8e5086`](https://github.com/WordPress/gutenberg/commit/d8e50867432ad92d5e3c7c618bc0235dd8adaf99)
- **Discussion:** [#83041](https://github.com/WordPress/gutenberg/pull/83041) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the block editor's rich-text focus-capture handler where touching a selection handle (hair pin) at the edge of a block on iOS would immediately clear the selection and cancel the drag, making it impossible to adjust selections in single-line paragraphs. The fix narrows the `contenteditable` toggle to only fire when the press target is inside a `[data-block]` element, so presses on the canvas around blocks no longer interfere with selection handles. Also clears a lingering collapsed caret in Safari after restoring `contenteditable`, and fixes a desktop triple-click issue where selecting a paragraph from the side left it inert to typing and deletion.

## Impact

- **Site owners / editors (mobile & Safari):** Selection handles at the edge of a block are now draggable on iOS; Safari no longer shows a ghost caret after clicking block padding; triple-clicking a paragraph from the side on desktop now allows typing to replace the selection. No configuration or code changes needed.
- **Plugin & theme developers:** No API, hook, or schema changes. The fix is entirely internal to `@wordpress/rich-text`'s `preventFocusCapture` hook. No action required.
- **Headless & REST consumers:** No impact.

## Technical details

The change is in `packages/rich-text/src/hook/event-listeners/prevent-focus-capture.js`, inside the `preventFocusCapture()` hook.

**Pointer-down guard (the core fix):**

```js
// Before: any press on an element containing the rich text triggered the toggle
if ( ! event.target.contains( element ) ) {
    return;
}
value = element.getAttribute( 'contenteditable' );
element.setAttribute( 'contenteditable', 'false' );
defaultView.getSelection().removeAllRanges();

// After: only acts when the press is on a block (e.g. group padding)
if ( ! event.target.contains( element ) ) {
    return;
}
if ( ! event.target.closest( '[data-block]' ) ) {
    return;
}
value = element.getAttribute( 'contenteditable' );
defaultView.getSelection().removeAllRanges();
element.setAttribute( 'contenteditable', 'false' );
```

The `closest('[data-block]')` check means a press on the canvas (outside any block) no longer makes the rich text non-editable, so a selection handle rendered in the gap between blocks stays interactive. The order of `removeAllRanges()` and `setAttribute` was also swapped (clear selection before toggling editability).

**Pointer-up Safari caret cleanup:**

```js
function onPointerUp() {
    if ( value !== null ) {
        element.setAttribute( 'contenteditable', value );
        value = null;
        // New: clear a collapsed caret Safari may have left inside the element
        const selection = defaultView.getSelection();
        if (
            selection.isCollapsed &&
            element.contains( selection.anchorNode )
        ) {
            selection.removeAllRanges();
        }
    }
}
```

**Test changes:**
- `test/e2e/specs/editor/various/multi-block-selection.spec.js` — the triple-click test now clicks the paragraph directly (via `getByRole('document').first().click({ clickCount: 3 })`) instead of clicking 5 px to the left on canvas padding. Added an assertion that typing `'x'` replaces the selected content.
- `test/e2e/specs/editor/various/rich-text.spec.js` — new test: inserts a `core/group` with flex layout and 60/40 px padding containing a paragraph, clicks the group's top padding, and asserts the group (not the inner paragraph) receives focus and that typing does not alter the paragraph content.
- `test/e2e/specs/editor/various/writing-flow.spec.js` — the "select text from the left edge of a block" test is now skipped in Chromium (`test.skip(browserName === 'chromium', …)`), because selection from outside a block into editable text was never supported in Chromium until #66402, and this PR restores the pre-#66402 behavior there. Added an assertion that the resulting selection is editable (typing replaces it).

## Contribution

Opened and merged by @ellatrix as a single-commit fix. The PR references #66402 (which introduced the original `contenteditable` toggle to stop Safari caret placement in flex-group padding) and #77136 (which added the "select from outside a block" e2e test now skipped in Chromium). CodeRabbit was invoked for review but produced no actionable comments. The PR description notes it was generated with Claude Code. No design debate or alternative approaches are visible in the 5-comment discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
