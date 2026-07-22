# #77333: Revisions screen with picker-activity layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] History`
- **Merged:** [`90b97b3`](https://github.com/WordPress/gutenberg/commit/90b97b3046f14694a2f34af9fb2ebba0152a8339)
- **Discussion:** [#77333](https://github.com/WordPress/gutenberg/pull/77333) · 12 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg editor now renders the revisions timeline inside the inspector controls using a `DataViewsPicker` with a `pickerActivity` layout. This replaces the previous static revision summary panels with a paginated, scrollable timeline that syncs with the revision slider and pins pagination to the bottom. The change improves navigation through revision history and aligns the revisions UI with the picker patterns used elsewhere in the editor.

## Impact

- **Plugin & theme developers:** The internal `RevisionAuthorPanel` and `RevisionCreatedPanel` components were removed. If your code imports or extends these, it will break. The revision summary area is now driven by `PostRevisionsTimeline`.
- **Editor sidebar customizers:** CSS targeting the old revision panels or relying on their DOM structure will need updating. The new timeline uses `DataViewsPicker` and scoped flex layouts (`:has(.editor-post-revisions-timeline)`) to manage height and sticky pagination.
- **Site owners & end users:** No action required. The "Open classic revisions screen" link remains functional.

## Technical details

- Introduces `packages/editor/src/components/post-revisions-timeline/index.js`, which uses `DataViewsPicker` with a `pickerActivity` layout to render revisions. It fetches revisions via `getPageRevisions` and `getCurrentRevision` from the `editor` store, and handles pagination/state synchronization with the revision slider.
- Replaces the old summary panels in `packages/editor/src/components/sidebar/post-revision-summary.js`:
  ```jsx
  // Before
  <RevisionAuthorPanel />
  <RevisionCreatedPanel />
  ```
  ```jsx
  // After
  <PostRevisionsTimeline />
  ```
- Adds `className` prop support to `RevisionDiffPanel` (`packages/editor/src/components/revision-diff-panel/index.js`) and applies `editor-revision-meta-diff__content` to cap the meta diff height at `80px` via new CSS in `post-revisions-timeline/style.scss`.
- The timeline uses `filterSortAndPaginate` from `@wordpress/dataviews` and syncs the active page when the slider selection changes, ensuring the selected revision stays visible.
- CSS scoped with `:has(.editor-post-revisions-timeline)` ensures the bounded flex layout only applies to the revisions inspector area, keeping pagination sticky at the bottom.

## Contribution

Opened by `@ntsekouras` as part of the history feature work, the PR explores a navigational picker layout for revisions to improve parity with Global Styles. Reviewers `@oandregal`, `@jameskoster`, and `@Mamaduka` discussed the choice of `DataViewsPicker` over `DataViews`, pagination behavior, and accessibility labeling. The team agreed to retain the "Open classic revisions screen" link until the new UI is fully polished, and deferred granular change summaries and revision naming to follow-ups. The implementation was merged with contributions from `@t-hamano`, `@jameskoster`, `@ramonjd`, `@talldan`, `@oandregal`, and `@Mamaduka`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
