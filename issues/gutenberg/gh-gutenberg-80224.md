# #80224: Notes: Allow canceling the autocompleter popover with Escape without dismissing the note form

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Editor`, `[Feature] Notes`
- **Merged:** [`645b4d9`](https://github.com/WordPress/gutenberg/commit/645b4d996198bcc6fa32050741647e9dd4a51828)
- **Discussion:** [#80224](https://github.com/WordPress/gutenberg/pull/80224) · 5 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug in the block notes editor where pressing Escape to dismiss the mention autocompleter popover also closed the note form. The change ensures that Escape only dismisses the note form when it hasn't already been handled by a child component, preserving the draft and improving the editing workflow for users adding notes with @ mentions.

## Impact

- **Site owners & editors:** Pressing Escape while the mention autocompleter is open now correctly dismisses only the popover, leaving the note form and draft intact.
- **Plugin & theme developers:** No action required. No public APIs, hooks, or block.json fields were changed.
- **Hosting & platform teams:** No action required.

## Technical details

The fix modifies the keydown handler in `packages/editor/src/components/collab-sidebar/note-form.js`. Previously, any Escape keypress triggered the `onCancel` callback, which dismissed the entire note form. The diff updates the condition to check `event.defaultPrevented`:

```diff
- if ( event.key === 'Escape' ) {
+ if ( event.key === 'Escape' && ! event.defaultPrevented ) {
```

This allows child components (like the mention autocompleter) to call `event.preventDefault()` when handling Escape, signaling that the keypress should not bubble up to cancel the form. An e2e test was added to `test/e2e/specs/editor/various/block-notes.spec.js` to verify the popover dismissal and form persistence.

## Contribution

Opened and merged by @Mamaduka, with testing and confirmation from @noruzzamans. The fix is a straightforward one-line condition update to respect `event.defaultPrevented`, avoiding the need for more complex event propagation handling. The PR was merged as commit `645b4d9`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
