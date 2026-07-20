# #80019: Notes: Add a "Resolved" divider above resolved notes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Notes`
- **Merged:** [`63899f5`](https://github.com/WordPress/gutenberg/commit/63899f5024b742cf159215539f04467485ea0678)
- **Discussion:** [#80019](https://github.com/WordPress/gutenberg/pull/80019) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a centered "Resolved" divider in the block editor's collaboration sidebar to visually separate unresolved and orphaned notes from resolved ones. The change also reorders the note list so orphaned notes (detached from deleted blocks) appear above the divider alongside active notes, rather than at the bottom. This improves scanability in the "All notes" view without altering the floating note view.

## Impact

- **Block editor users & collaboration teams**: Improved visual hierarchy in the Notes sidebar; no configuration or migration required.
- **Plugin & theme developers**: No public API changes or breaking changes. Custom editor UI overrides targeting `.editor-collab-sidebar-panel` may need to account for the new `.editor-collab-sidebar-panel__status-separator` class if they rely on exact DOM structure or CSS specificity.
- **No action required** for standard site builds or external plugins.

## Technical details

- `packages/editor/src/components/collab-sidebar/hooks.js`: The `useNoteThreads` hook now returns notes in the order `[...unresolved, ...orphans, ...resolved]` instead of placing orphans last.
- `packages/editor/src/components/collab-sidebar/notes.js`: Calculates `firstResolvedIndex` by finding the first thread where `thread.status === 'approved'` and `thread.blockClientId` exists. Wraps the `threads.map()` output in a `Fragment` and conditionally renders a `<Stack>` containing a `<Text variant="heading-sm">` labeled "Resolved" at that index. The floating view (`isFloating`) skips this divider.
- `packages/editor/src/components/collab-sidebar/style.scss`: Introduces `.editor-collab-sidebar-panel__status-separator`, applying flexbox-based horizontal lines via `::before`/`::after` pseudo-elements, mirroring the existing `__more-reply-separator` pattern.
- E2E tests in `test/e2e/specs/editor/various/block-notes.spec.js` verify the divider's visibility, text content, and DOM ordering relative to unresolved, orphaned, and resolved notes.

## Contribution

Opened by @adamsilverstein and merged in commit `63899f5`. The implementation builds on a proof of concept by @Mamaduka and follows design markup from @jasmussen. Review feedback from @jeffpaul and @adamsilverstein refined the sorting logic to keep orphaned notes above the divider, and scoped the change to the minimal consensus subset, deferring broader structural changes (like a separate "Detached" section) to follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
