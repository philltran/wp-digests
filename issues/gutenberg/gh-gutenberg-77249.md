# #77249: Show media upload progress in a snackbar

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Status] In Progress`, `Needs Design Feedback`, `[Package] Editor`, `Needs Accessibility Feedback`, `[Package] Edit Post`, `[Type] Feature`, `[Package] Edit Site`, `[Feature] Client Side Media`
- **Merged:** [`26abed4`](https://github.com/WordPress/gutenberg/commit/26abed4b30afea20b3956756f514a48e0edfef05)
- **Discussion:** [#77249](https://github.com/WordPress/gutenberg/pull/77249) · 21 comments · 1 reactions

## Summary

A new `UploadProgressSnackbar` component has been merged into the WordPress block editor to provide a persistent, centralized indicator for media upload progress in both the post and site editors. The snackbar displays a spinner during uploads and shows a completion checkmark once the queue drains, automatically disappearing when idle. It covers both client-side media processing (CSM) and traditional server-upload paths, ensuring users see consistent feedback during batch uploads regardless of browser support or configuration.

## Impact

- **Theme & Plugin Developers**: No action required. The component is internal to `@wordpress/editor` and does not expose public filters, hooks, or render props for external customization.
- **Editor/Site Editors**: Automatically active in modern browsers where CSM is enabled. For environments where the `wp_client_side_media_processing_enabled` filter returns false or browser support is lacking (e.g., Safari), progress tracking falls back to the editor-local tracker populated by the `mediaUpload` wrapper.
- **Accessibility/Screen Reader Users**: The notice triggers `wp.a11y.speak()` exactly once on upload start and once on completion (`'Media upload started'`, `'Media upload complete'`). Per-tick progress updates do not trigger screen reader announcements because the notices store dispatch uses `speak: false`.

## Technical details

- **New Component**: `packages/editor/src/components/upload-progress-snackbar/index.js` contains the primary controller. It mounts invisibly and manages a persistent notice via the `noticesStore`.
- **Registration**: Added to both layout files:
  - `packages/edit-post/src/components/layout/index.js`
  - `packages/edit-site/src/components/layout/index.js`
  Both now import `UploadProgressSnackbar` from `@wordpress/editor` and render it as a sibling to `<SnackbarNotices>`.
- **Data Sources**:
  - **CSM Path**: Subscribes to `uploadStore.getItems()` from `@wordpress/upload-media`. Filters by `!parentId` to count only original uploads, excluding generated subsizes/thumbnails.
  - **Non-CSM Path**: Uses an editor-local tracker (`./tracker.js`) exposing `addFiles()` and `advance()`. The `mediaUpload` wrapper (`packages/editor/src/utils/media-upload/index.js`) calls these methods during batch start, file completion, and error states.
- **State Management**: Relies on a constant `NOTICE_ID = 'upload-progress'`. Replacing the notice with the same ID updates text in-place without exit/enter animation flicker. A `peak` state (tracked via `useRef`) maintains a high-water mark for batch counts across both sources.
- **Lifecycle**: Displays a spinner while uploads are active. When all items drain, it replaces the notice with a completion checkmark (`'Upload complete'`) for `COMPLETION_DISPLAY_MS` (3000ms) before calling `removeNotice( NOTICE_ID )`.
- **Accessibility Styling**: `style.scss` resets default Spinner margins and sets `fill: currentColor` on the checkmark SVG to ensure vertical centering and proper contrast within the dark snackbar theme.

## Contribution

Opened by `@adamsilverstein` to close #77241 after identifying that batch uploads only showed per-block spinners without centralized progress feedback. Initial implementations were gated strictly to client-side media processing (CSM), but the author extended support to the traditional server-upload path before merge. Design and accessibility reviews from `@jasmussen`, `@andrewserong`, and `@joedolson` refined the visual treatment (spinner/checkmark alignment) and confirmed that single-transition screen reader announcements (`wp.a11y.speak()`) avoided per-tick chatter. The PR merged with co-authors including `@mamaduka`, `@karmatosed`, and `@andrewserong`, adding ~1.13 kB to the editor bundle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
