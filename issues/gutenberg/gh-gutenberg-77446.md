# #77446: Feature: Need to add “Show More” / “Show Less” toggle in Note.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @hbhalodia
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] Editor`, `[Feature] Notes`
- **Merged:** [`fda9287`](https://github.com/WordPress/gutenberg/commit/fda92876662bc25efbee89898a07377a6d42555d)
- **Discussion:** [#77446](https://github.com/WordPress/gutenberg/pull/77446) · 16 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The collaborative notes sidebar now automatically collapses long comments and displays a "Show more" / "Show less" toggle button. This prevents the sidebar from becoming excessively tall when users paste large blocks of text into a note, improving layout stability and readability in the block editor.

## Impact

- **Site owners & editors:** No action required. Long notes in the collaborative sidebar will now truncate to three lines by default, with a toggle to expand them.
- **Plugin & theme developers:** No action required. This is an internal editor UI change with no public API surface or deprecations.
- **Hosting & platform teams:** No configuration or migration needed. The behavior ships as a default client-side update.
- **Headless & REST consumers:** Unaffected. The change is purely a Gutenberg editor component enhancement.

## Technical details

The change modifies `packages/editor/src/components/collab-sidebar/note.js` to detect content overflow using `scrollHeight` and `clientHeight` inside a `useEffect` hook. When overflow is detected, it sets an `isOverflowing` state and applies the `is-collapsed` class to the `.editor-collab-sidebar-panel__note-content` container. The corresponding CSS in `style.scss` uses `-webkit-line-clamp: 3` and `display: -webkit-box` to truncate the text. A toggle button imported from `@wordpress/ui` (`variant="unstyled"`) is rendered when `isOverflowing` is true and the note is not in edit mode. The collapse state resets to `true` whenever `note?.content?.raw` changes, ensuring edited notes always render in full. No new hooks, filters, or REST schema changes are introduced.

## Contribution

Opened by `@hbhalodia` to address issue #72822, the PR initially proposed a basic toggle. During review, `@jasmussen` provided design feedback requesting a smaller, sentence-cased button that aligns with existing design tokens. `@ciampo` and `@jameskoster` steered the implementation toward using the `unstyled` variant from `@wordpress/ui` to match the desired semantics and spacing without introducing extra padding. The author iterated on edge cases—such as forcing expanded state during edits and hiding the toggle when content fits—before the PR was merged with several co-authors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
