# #81064: Notes: Display shortcut for form action buttons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Notes`
- **Merged:** [`e1f2c2c`](https://github.com/WordPress/gutenberg/commit/e1f2c2cec8c9ae1a222ad66a13329b3ed87c3cfb)
- **Discussion:** [#81064](https://github.com/WordPress/gutenberg/pull/81064) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor now displays keyboard shortcut hints on the Notes form action buttons in the collaboration sidebar. Pressing `Escape` cancels the note draft, while `⌘+Enter` (or `Ctrl+Enter` on Windows/Linux) submits it. This improves discoverability and workflow speed for editors using the collaborative notes feature.

## Impact

- **Editor users & content creators:** No direct action required; the UI now shows shortcut hints for the Notes form.
- **Plugin & theme developers:** No impact. This is an internal editor UI change.
- **Headless & REST consumers:** No impact.
- **No action required** for site owners or developers.

## Technical details

The change modifies `packages/editor/src/components/collab-sidebar/note-form.js` to import `displayShortcut` from `@wordpress/keycodes` and pass it to the `Button` components in the `NoteForm` component. The cancel button receives `shortcut="Escape"`, and the submit button receives `shortcut={ displayShortcut.primary( 'Enter' ) }`. This leverages the existing `@wordpress/ui` `Button` component's `shortcut` prop to render platform-appropriate keyboard hints without altering the form's submission logic or event handlers.

Before/after usage pattern:
```diff
- <Button size="compact" variant="tertiary" onClick={ onCancel }>
+ <Button
+     size="compact"
+     variant="tertiary"
+     onClick={ onCancel }
+     shortcut="Escape"
+ >
```

## Contribution

Opened and merged by @Mamaduka, with UI/UX review from @jasmussen. The discussion centered on the visual formatting of the macOS shortcut hint, with @jasmussen suggesting `⌘ + Enter` over `⌘Enter`. @Mamaduka clarified that the current format aligns with existing shortcut display conventions in the editor, and the change was approved and merged without further iteration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
