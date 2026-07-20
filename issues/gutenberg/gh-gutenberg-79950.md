# #79950: Visual revisions: Label autosaves in the revisions timeline

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @priethor
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] History`
- **Merged:** [`232699d`](https://github.com/WordPress/gutenberg/commit/232699d1b6e59033291846118c637f1a7f817f57)
- **Discussion:** [#79950](https://github.com/WordPress/gutenberg/pull/79950) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds an `Autosave` badge to autosave entries in the block editor’s visual revisions timeline. The label appears next to the revision date and is appended to the row’s accessible name, improving visual scannability and screen-reader context. This aligns the new timeline with the classic revisions screen’s behavior.

## Impact

- **Site owners & content editors:** Autosave entries in the visual revisions timeline now display a distinct `Autosave` badge and include the label in the row’s accessible name.
- **Plugin & theme developers:** No breaking changes, removed APIs, or new public hooks. No action required.

## Technical details

In `packages/editor/src/components/post-revisions-timeline/index.js`, the component now uses `isAutosaveRevision( item )` to detect autosaves by checking if `item.slug?.endsWith( '-autosave-v1' )`. A new `RevisionBadges` component renders a `<Badge intent="none">Autosave</Badge>` inside a `<Stack>` alongside the date `Text`. The `getValue` callback for the date field now uses `sprintf( __( '%1$s, %2$s' ), displayDate, __( 'Autosave' ) )` to append the label to the accessible name. The component also removes the `currentRevision` selector and `postContent` state, switching `PostContentInformationUI` to read `item.content?.raw` directly from the timeline item. To support this, `packages/editor/src/store/private-selectors.js` adds `'slug'` to the fields requested in `buildRevisionsPageQuery`.

## Contribution

Opened and merged by @priethor as part of the visual revisions initiative (#79120). During review, @Mamaduka suggested deriving the autosave status client-side via the revision slug rather than modifying the REST API, which the author adopted. @jasmussen also contributed to the merge. The change ships with a dedicated unit test suite for the timeline component.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
