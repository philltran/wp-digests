# #80314: Visual revisions: Add a code diff view inside the editor

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @priethor
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] History`
- **Merged:** [`bb5f06c`](https://github.com/WordPress/gutenberg/commit/bb5f06cb267c138b3f1b680cdb4ee5b929d0436b)
- **Discussion:** [#80314](https://github.com/WordPress/gutenberg/pull/80314) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The block editor now includes a code diff view in the revisions screen, allowing users to inspect raw block markup changes between revisions. A new "Code editor" option in the revisions options menu toggles between the visual canvas and a line-by-line diff highlighting added and removed markup. This addresses cases where visual diffs fail to surface changes to block attributes that do not alter the rendered output.

## Impact

- **Site owners & editors**: Gains a new markup-level audit tool for revisions without leaving the block editor.
- **Plugin & theme developers**: No breaking changes or removed APIs. Developers extending the revisions UI should note the new `isRevisionMode` prop on `MoreMenu` and the conditional rendering logic in `EditorInterface`.
- **Hosting & platform teams**: No action required.
- **Headless & REST consumers**: No impact; this is a purely client-side editor UI change.

## Technical details

- Adds `packages/editor/src/components/post-revisions-preview/revisions-code-diff.js`, which uses `diffLines` from the `diff` package to compute line-level changes between `previousRevision.content.raw` and `revision.content.raw`.
- `EditorInterface` now conditionally renders `<RevisionsCodeDiff />` when `mode === 'text'`, falling back to `<RevisionsCanvas />` otherwise.
- `MoreMenu` accepts a new `isRevisionMode` prop; when true, it renders a `ModeSwitcher` dropdown instead of the full tools/view menu.
- `RevisionsHeader` passes `isRevisionMode` to `MoreMenu` in revision context.
- `RevisionsSlider` is refactored into a data-fetching wrapper (`ConnectedRevisionsSlider`) and a presentational component, with improved focus management using `useFocusOnMount` and `useRef` to prevent focus jumps during network throttling.
- CSS is added in `style.scss` for `.editor-revisions-code-diff` and its table rows, including line-number and status styling.

## Contribution

Opened and merged by @priethor with co-authors @jorgefilipecosta, @ellatrix, @himanshupathak95, and @simison. The PR description notes that @priethor guided the implementation using Codex, with the team reviewing the diff logic, focus behavior, and testing instructions. Discussion was minimal, focusing on implementation details and edge-case handling for pagination and loading states.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
