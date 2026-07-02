# #78218: Notes: inline (partial-text) notes via hybrid marker + strip-on-render approach

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Package] Editor`, `[Type] Feature`, `[Feature] Notes`
- **Merged:** [`dc9c6db`](https://github.com/WordPress/gutenberg/commit/dc9c6dbc2c5b0a821d40682646d4360de83c4b3b)
- **Discussion:** [#78218](https://github.com/WordPress/gutenberg/pull/78218) · 38 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Introduces inline (partial-text) annotations to the Block Editor by anchoring notes directly to selected text within a block, rather than only to whole blocks. The implementation uses a hybrid approach: a `core/note` rich-text marker survives content edits in the editor, while an on-render PHP filter strips the wrapper from frontend output to keep HTML clean. This enables resilient cross-edit annotations and establishes a forward-compatible foundation for CRDT/Yjs real-time collaboration.

## Impact

['- **Block & Editor Plugin Developers:** Can now interact with or extend the `core/note` rich-text format. The new `render_block`-attached filter (`gutenberg_strip_inline_note_markers`) may need consideration if custom note rendering or block output modification is implemented.', '- **Theme & Platform Developers:** No action required. Frontend HTML remains free of annotation markers, and legacy content without inline anchors falls back safely to block-level notes or stored selection metadata.']

## Technical details

The diff registers a new rich-text format (`core/note`) in `packages/editor/src/components/collab-sidebar/format.js` with `tagName: 'mark'` and `className: 'wp-note'`. When `useNoteActions.onCreate` captures a non-collapsed selection, it applies the format to the targeted RichTextData attribute. Anchoring resilience is handled by `findNoteRange`, which re-scans the rich-text record for the marker after edits, with `_wp_note_selection` comment meta stored as a fallback. On the PHP side, `lib/compat/wordpress-7.1/block-comments.php` hooks `gutenberg_strip_inline_note_markers()` to `render_block`. It instantiates an anonymous subclass of `WP_HTML_Tag_Processor` to locate `<mark class="wp-note">`, tracks nesting via a stack, and removes open/closer tokens while preserving inner text. This ensures the raw `post_content` retains markers for editor reloads while the rendered frontend stays clean.

## Contribution

Merged as PR #78218, authored by @adamsilverstein with co-authors including @Mamaduka and @youknowriad. The PR consolidated two earlier explorations (annotation-based decoration vs. rich-text anchoring) into a hybrid model that preserves marker position across edits while stripping it during rendering. It was merged as part of the WordPress 7.1 notes milestone (#59445), building on the multi-note-per-block array shape from #75147.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
